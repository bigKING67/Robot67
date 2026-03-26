# Tool Design Guidelines

> ACI principles and data-specific tool patterns for this Agent.

---

## Overview

Tools are the Agent's capabilities. This project follows **ACI (Agent-Computer Interface)** design principles from the reference article: tools should match the Agent's goals, not low-level API operations.

**Critical rule: Debug tool selection errors by checking tool descriptions first, not model capability.**

---

## Tool Categories

### 1. Built-in Tools (TypeScript, in Agent Server)

Basic Agent capabilities, always available:

| Tool | Purpose |
|------|---------|
| `read` | Read file contents |
| `write` | Write file contents |
| `edit` | Edit file with diff |
| `shell` | Execute shell commands |

### 2. Data Tools (Python, via MCP)

Exposed by the Data Analysis MCP Server:

| Tool | Purpose | When Agent Should Use |
|------|---------|---------------------|
| `query_database` | Execute SQL on PostgreSQL/DuckDB | When user asks about specific data |
| `load_data` | Load CSV/Excel/Parquet into DuckDB | When analyzing local files |
| `compute_metrics` | Calculate YoY, MoM, mean, percentiles | When comparing time periods |
| `detect_anomaly` | Statistical anomaly detection | When checking if a metric is abnormal |
| `drill_down` | Multi-dimensional breakdown | When finding which dimension caused a change |
| `funnel_analysis` | Conversion funnel analysis | When analyzing user conversion steps |
| `cohort_analysis` | Retention / cohort analysis | When analyzing user retention |
| `attribution_analysis` | Cause attribution | When determining why a metric changed |
| `search_industry` | Search industry data/reports | When needing external market context |
| `search_competitor` | Search competitor info | When comparing with competitors |
| `search_news` | Search industry news | When needing recent market events |
| `generate_action_plan` | Create executable action checklist | When giving recommendations |
| `design_ab_test` | Design A/B test plan | When suggesting experiments |
| `estimate_roi` | ROI estimation | When forecasting impact |

### 3. External MCP Tools

Connected via MCP Client Hub:

| MCP Server | Tools | When |
|-----------|-------|------|
| exa | web_search, get_code_context | Industry research, competitor search |
| Future: Slack/Feishu | send_message | Alert notifications |
| Future: Notion | create_page | Report archival |

---

## ACI Design Principles

### 1. Tool = Agent's Goal, Not API Operation

```python
# BAD — too low-level, maps to API operation
{
    "name": "execute_sql",
    "description": "Execute a SQL query"
}

# GOOD — maps to Agent's analytical goal
{
    "name": "drill_down",
    "description": "Break down a metric by multiple dimensions to find which "
        "dimension contributed most to the change. Use when a KPI increased or "
        "decreased and you need to find which channel/region/segment caused it. "
        "Returns ranked dimensions with contribution percentages.",
    "inputSchema": {
        "metric": "The metric to analyze (e.g., 'revenue', 'conversion_rate')",
        "dimensions": "Dimensions to break down by (e.g., ['channel', 'region'])",
        "time_range": "Time period to analyze",
        "comparison": "Comparison period (e.g., 'previous_week')"
    }
}
```

### 2. Descriptions Must Be Routing Conditions

Tool descriptions should read like "Use when X / Don't use when Y":

```python
{
    "name": "detect_anomaly",
    "description": "Check if a metric value is statistically anomalous compared "
        "to its historical baseline. Use when the user asks 'is this normal?' or "
        "when a metric change seems unusual. Don't use for simple comparisons — "
        "use compute_metrics instead. Returns: is_anomalous (bool), z_score, "
        "expected_range, severity.",
}
```

### 3. Structured Error with Fix Suggestion

```python
# Tool returns structured error, not exception
{
    "success": False,
    "error": {
        "code": "INVALID_METRIC",
        "message": "Metric 'gmv' not found in database",
        "suggestion": "Available metrics: revenue, order_count, user_count. "
            "Did you mean 'revenue'?"
    }
}
```

### 4. Return Structured Data, Not Formatted Text

```python
# BAD — formatted text (hard for Agent to reason about)
"Revenue decreased 15% this week"

# GOOD — structured data (Agent formats it for the user)
{
    "metric": "revenue",
    "current": 1500000,
    "previous": 1764706,
    "change_pct": -15.0,
    "is_anomalous": True,
    "z_score": -2.3,
    "top_contributors": [
        {"dimension": "channel", "value": "organic", "contribution_pct": 62.0},
        {"dimension": "channel", "value": "paid_search", "contribution_pct": 28.0}
    ]
}
```

---

## Tool Schema Validation

All tool inputs/outputs validated with schemas:

- **TypeScript tools**: zod
- **Python MCP tools**: pydantic

```python
# Python MCP tool with pydantic validation
from pydantic import BaseModel, Field

class DrillDownInput(BaseModel):
    metric: str = Field(description="Metric name to analyze")
    dimensions: list[str] = Field(description="Dimensions to break down by")
    time_range: str = Field(description="Time range, e.g. 'last_7_days'")
    comparison: str = Field(default="previous_period", description="Comparison period")

class DrillDownOutput(BaseModel):
    metric: str
    total_change: float
    breakdown: list[DimensionContribution]
    recommendation: str
```

---

## Forbidden Patterns

| Pattern | Why | Use Instead |
|---------|-----|-------------|
| Vague description ("do stuff") | LLM picks wrong tool | Specific routing condition |
| Returning formatted text | Agent can't reason about it | Return structured JSON |
| Throwing exceptions | Agent loop breaks | Return structured error |
| Exposing raw SQL as tool | Security risk, bad UX | Domain-specific analysis tools |
| One mega-tool ("analyze everything") | LLM can't learn when to use it | Focused tools per goal |

---

## Common Mistakes

1. **Tool description too vague** — #1 cause of tool selection errors
2. **Too many tools** — 5 MCP servers = ~55K tokens of definitions; keep tools focused
3. **Missing "Don't use when"** — Causes tool confusion between similar tools
4. **Returning raw DataFrames** — Always summarize; LLM context is limited
