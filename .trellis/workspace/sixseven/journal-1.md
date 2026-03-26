# Journal - sixseven (Part 1)

> AI development session journal
> Started: 2026-03-26

---



## Session 1: Bootstrap Guidelines: Data Analysis Agent Architecture

**Date**: 2026-03-26
**Task**: Bootstrap Guidelines: Data Analysis Agent Architecture

### Summary

(Add summary)

### Main Changes

## Summary

Initialized project with complete development guidelines for a business-growth data analysis Agent.

## Architecture Decided

| Layer | Technology | Role |
|-------|-----------|------|
| Frontend | Vite + React + ECharts | Chat UI, Dashboard, Report Generator (SPA) |
| Agent Server | TypeScript (Bun) | Agent Loop, MCP Client, Session management |
| Performance Modules | Rust (NAPI-RS) | Vector search, embeddings, concurrent scheduling |
| Data Analysis MCP | Python (polars + scipy) | SQL query, metrics, anomaly detection, drill-down |
| Business DB | PostgreSQL + DuckDB | Business data + OLAP analysis engine |

## Reference Projects Analyzed

- **pi-mono** (badlogic) — Agent Loop events, JSONL sessions, Skills, context compaction
- **ironclaw** (nearai) — Rust ToolRegistry, typed errors, WASM sandbox, hybrid search
- **cortex** (rikouu) — 3-layer memory, BM25+Vector RRF, memory lifecycle
- **universal-db-mcp** (Anarkh-Lee) — MCP adapter pattern, structured errors, zod validation

## Spec Files Written (15 files, 2086 lines)

**Backend (8 files)**:
- `directory-structure.md` — Polyglot monorepo: agent-server/ + data-mcp/ + web/
- `database-guidelines.md` — PostgreSQL + DuckDB + SQLite + JSONL patterns
- `error-handling.md` — Typed error hierarchy (TS + Rust + Python)
- `logging-guidelines.md` — pino(TS) + tracing(Rust) + structlog(Py)
- `quality-guidelines.md` — Per-language lint/test standards
- `tool-design-guidelines.md` — ACI principles, data analysis tool patterns
- `data-engine-guidelines.md` — Python MCP, polars (not pandas), analysis patterns

**Frontend (7 files)**:
- `directory-structure.md` — Vite React SPA: Chat/Dashboard/Reports
- `component-guidelines.md`, `hook-guidelines.md`, `state-management.md`, `type-safety.md`, `quality-guidelines.md`

## Key Design Decisions

1. **Bun not Node.js** — 3-5x faster, better for Agent prompt/tool iteration speed
2. **polars not pandas** — 10-100x faster, Rust backend, lazy evaluation
3. **DuckDB not SQLite** — Columnar OLAP, 100-1000x faster for analytics
4. **Agent produces data, frontend renders charts** — ECharts > matplotlib
5. **MCP decouples languages** — Each component can be independently replaced


### Git Commits

| Hash | Message |
|------|---------|
| `713d0cb` | (see git log) |

### Testing

- [OK] (Add test results)

### Status

[OK] **Completed**

### Next Steps

- None - task complete
