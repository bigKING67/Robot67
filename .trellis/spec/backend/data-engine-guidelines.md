# Data Engine Guidelines

> Python Data Analysis MCP Server patterns and conventions.

---

## Overview

The Data Analysis MCP Server is the Agent's analytical brain. It runs as a standalone Python process, communicating with the Agent Server via MCP protocol.

**Stack**: Python 3.12+ · polars (not pandas) · scipy · DuckDB · PostgreSQL · pydantic · MCP SDK

---

## Why polars Over pandas

| Aspect | polars | pandas |
|--------|--------|--------|
| Performance | 10-100x faster (Rust backend) | Slow for large datasets |
| Memory | Lazy evaluation, streaming | Eager, loads all into memory |
| API | Consistent, expressive | Inconsistent, many gotchas |
| Type safety | Strict column types | Loose, silent type coercion |
| Parallelism | Multi-threaded by default | Single-threaded (GIL) |

```python
# polars example — lazy evaluation chain
import polars as pl

result = (
    pl.scan_csv("sales.csv")
    .filter(pl.col("date") >= "2026-03-01")
    .group_by("channel")
    .agg([
        pl.col("revenue").sum().alias("total_revenue"),
        pl.col("orders").sum().alias("total_orders"),
        (pl.col("revenue").sum() / pl.col("orders").sum()).alias("avg_order_value"),
    ])
    .sort("total_revenue", descending=True)
    .collect()  # Executes the full query plan at once
)
```

---

## Database Adapters

### PostgreSQL (business data)

```python
import asyncpg

class PostgresAdapter:
    def __init__(self, dsn: str):
        self.pool = None

    async def connect(self):
        self.pool = await asyncpg.create_pool(self.dsn, min_size=2, max_size=10)

    async def query(self, sql: str, params: list | None = None) -> pl.DataFrame:
        async with self.pool.acquire() as conn:
            rows = await conn.fetch(sql, *(params or []))
            return pl.DataFrame([dict(r) for r in rows])
```

### DuckDB (local analysis engine)

```python
import duckdb

class DuckDBAdapter:
    def __init__(self, db_path: str = ":memory:"):
        self.conn = duckdb.connect(db_path)

    def query(self, sql: str) -> pl.DataFrame:
        return self.conn.execute(sql).pl()  # Native polars output

    def load_csv(self, path: str, table_name: str) -> None:
        self.conn.execute(f"CREATE TABLE {table_name} AS SELECT * FROM '{path}'")

    def load_parquet(self, path: str, table_name: str) -> None:
        self.conn.execute(f"CREATE TABLE {table_name} AS SELECT * FROM '{path}'")
```

**Use DuckDB for**:
- CSV/Excel/Parquet file analysis
- Complex OLAP queries on loaded data
- Cross-source joins (PG data + CSV data)

**Use PostgreSQL for**:
- Live business data queries
- Transactional data access

---

## Analysis Tool Patterns

### Metrics Computation

```python
def compute_metrics(
    df: pl.DataFrame,
    metric_col: str,
    time_col: str,
    period: str = "week",
) -> dict:
    current = df.filter(pl.col(time_col) >= current_period_start)
    previous = df.filter(
        (pl.col(time_col) >= previous_period_start)
        & (pl.col(time_col) < current_period_start)
    )

    current_val = current[metric_col].sum()
    previous_val = previous[metric_col].sum()

    return {
        "current": current_val,
        "previous": previous_val,
        "change": current_val - previous_val,
        "change_pct": ((current_val - previous_val) / previous_val * 100)
            if previous_val != 0 else None,
        "period": period,
    }
```

### Anomaly Detection

```python
from scipy import stats

def detect_anomaly(
    values: list[float],
    current_value: float,
    method: str = "zscore",
    threshold: float = 2.0,
) -> dict:
    if method == "zscore":
        mean = np.mean(values)
        std = np.std(values)
        z_score = (current_value - mean) / std if std > 0 else 0

        return {
            "is_anomalous": abs(z_score) > threshold,
            "z_score": round(z_score, 2),
            "mean": round(mean, 2),
            "std": round(std, 2),
            "expected_range": [
                round(mean - threshold * std, 2),
                round(mean + threshold * std, 2),
            ],
            "severity": "high" if abs(z_score) > 3 else "medium" if abs(z_score) > 2 else "low",
        }
```

### Multi-Dimensional Drill-Down

```python
def drill_down(
    df: pl.DataFrame,
    metric_col: str,
    dimensions: list[str],
    current_period: tuple,
    previous_period: tuple,
) -> list[dict]:
    """Find which dimension values contributed most to metric change."""
    results = []

    for dim in dimensions:
        current = (
            df.filter(pl.col("date").is_between(*current_period))
            .group_by(dim)
            .agg(pl.col(metric_col).sum().alias("current"))
        )
        previous = (
            df.filter(pl.col("date").is_between(*previous_period))
            .group_by(dim)
            .agg(pl.col(metric_col).sum().alias("previous"))
        )

        joined = current.join(previous, on=dim, how="outer")
        joined = joined.with_columns(
            ((pl.col("current") - pl.col("previous")) / pl.col("previous") * 100)
            .alias("change_pct")
        )

        top_movers = joined.sort("change_pct").head(5)
        results.append({
            "dimension": dim,
            "top_movers": top_movers.to_dicts(),
        })

    return results
```

---

## MCP Server Structure

```python
# server.py — MCP entry point
from mcp.server import Server
from mcp.types import Tool

app = Server("data-analysis-mcp")

@app.tool()
async def query_database(sql: str, database: str = "postgres") -> dict:
    """Execute SQL query on business database.
    Use when the user asks about specific data points or needs raw data.
    Don't use for analysis — use analysis tools instead."""
    adapter = get_adapter(database)
    df = await adapter.query(sql)
    return {"columns": df.columns, "rows": df.to_dicts(), "row_count": len(df)}

@app.tool()
async def drill_down(metric: str, dimensions: list[str], time_range: str) -> dict:
    """Break down a metric by dimensions to find root cause of changes.
    Use when a KPI changed and you need to find which segment caused it."""
    # ... implementation
```

---

## Forbidden Patterns

| Pattern | Why | Use Instead |
|---------|-----|-------------|
| `import pandas as pd` | Slow, memory-hungry | `import polars as pl` |
| Raw SQL string formatting | SQL injection | Parameterized queries |
| Returning full DataFrame to Agent | Exceeds context limit | Return summary + top N rows |
| `plt.savefig()` for charts | Agent produces data, frontend renders charts | Return structured chart data |
| Blocking sync DB calls | Blocks event loop | Use async adapters (asyncpg) |

---

## Common Mistakes

1. **Returning too much data** — LLM context is limited; always summarize and return top N
2. **Using pandas** — polars is 10-100x faster; there's no reason to use pandas in new code
3. **Generating images** — The frontend renders charts from structured data; MCP returns JSON, not PNGs
4. **Missing parameterized queries** — Always use query parameters, never f-strings for SQL
5. **Not handling empty results** — Always check for empty DataFrames before computing metrics
