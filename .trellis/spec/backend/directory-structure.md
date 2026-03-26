# Directory Structure

> How backend code is organized in this project.

---

## Overview

This project is a **polyglot monorepo** for a business-growth data analysis Agent. Three languages serve different roles:

- **TypeScript (Bun)** — Agent Server: loop orchestration, MCP client, session management
- **Rust** — Performance modules: vector search, embedding compute, concurrency
- **Python** — Data Analysis MCP Server: querying, statistical analysis, visualization data

Architecture references: [pi-mono](https://github.com/badlogic/pi-mono) (Agent Loop), [ironclaw](https://github.com/nearai/ironclaw) (Rust patterns), [cortex](https://github.com/rikouu/cortex) (memory), [universal-db-mcp](https://github.com/Anarkh-Lee/universal-db-mcp) (MCP adapter).

---

## Directory Layout

```
dataagent/
├── agent-server/             # TypeScript (Bun) — Agent core
│   ├── src/
│   │   ├── agent/                # Agent Loop (pi-mono pattern)
│   │   │   ├── loop.ts              # Perceive → Decide → Act → Feedback
│   │   │   ├── session.ts           # Multi-user session management
│   │   │   ├── context.ts           # Context compaction + branch summarization
│   │   │   ├── dispatcher.ts        # Tool call dispatcher
│   │   │   └── events.ts            # Agent lifecycle events
│   │   │
│   │   ├── tools/                # Built-in tools (pi-mono basics)
│   │   │   ├── registry.ts          # ToolRegistry (ironclaw pattern)
│   │   │   ├── read.ts
│   │   │   ├── write.ts
│   │   │   ├── edit.ts
│   │   │   └── shell.ts
│   │   │
│   │   ├── mcp/                  # MCP Client Hub
│   │   │   ├── client.ts            # MCP client manager
│   │   │   └── discovery.ts         # Auto-discover MCP servers
│   │   │
│   │   ├── llm/                  # LLM Provider abstraction
│   │   │   ├── provider.ts          # LlmProvider interface
│   │   │   ├── registry.ts          # ModelRegistry
│   │   │   └── providers/
│   │   │       ├── anthropic.ts
│   │   │       └── openai.ts
│   │   │
│   │   ├── memory/               # Memory system (cortex pattern)
│   │   │   ├── store.ts             # Memory store interface
│   │   │   ├── layers.ts            # working / core / archive
│   │   │   └── search.ts            # Hybrid search (BM25 + vector)
│   │   │
│   │   ├── skills/               # Skills loader (pi-mono pattern)
│   │   │   ├── loader.ts            # Lazy load skill definitions
│   │   │   └── index.ts
│   │   │
│   │   ├── server/               # HTTP/WebSocket gateway
│   │   │   ├── app.ts               # Bun HTTP server
│   │   │   ├── ws.ts                # WebSocket for streaming
│   │   │   └── routes/
│   │   │       ├── chat.ts           # Chat API
│   │   │       ├── sessions.ts       # Session management API
│   │   │       └── reports.ts        # Report generation API
│   │   │
│   │   ├── safety/               # Security layer
│   │   │   ├── validator.ts
│   │   │   └── sanitizer.ts
│   │   │
│   │   └── types.ts              # Shared type definitions
│   │
│   ├── native/                   # Rust NAPI-RS modules
│   │   ├── src/
│   │   │   ├── lib.rs                # NAPI entry point
│   │   │   ├── vector_search.rs      # Vector similarity search
│   │   │   ├── embeddings.rs         # Embedding compute
│   │   │   └── scheduler.rs          # Concurrent session scheduler
│   │   ├── Cargo.toml
│   │   └── build.rs
│   │
│   ├── skills/                   # Skill definition files
│   │   ├── daily-monitor.md          # Daily metric monitoring
│   │   ├── root-cause.md            # Root cause analysis
│   │   ├── campaign-review.md       # Campaign review
│   │   └── budget-allocation.md     # Budget allocation
│   │
│   ├── package.json
│   ├── tsconfig.json
│   └── bunfig.toml               # Bun configuration
│
├── data-mcp/                     # Python — Data Analysis MCP Server
│   ├── src/
│   │   ├── server.py                 # MCP server entry point
│   │   ├── tools/
│   │   │   ├── registry.py           # Tool registration
│   │   │   ├── data/                 # Data querying tools
│   │   │   │   ├── query.py              # SQL execution (PG + DuckDB)
│   │   │   │   ├── loader.py             # CSV/Excel/Parquet loader
│   │   │   │   └── adapters/             # Database adapters
│   │   │   │       ├── postgres.py
│   │   │   │       └── duckdb_adapter.py
│   │   │   │
│   │   │   ├── analysis/             # Analysis tools (core differentiator)
│   │   │   │   ├── metrics.py            # YoY, MoM, mean, percentile
│   │   │   │   ├── anomaly.py            # Anomaly detection (Z-score, IQR)
│   │   │   │   ├── drilldown.py          # Multi-dim drill-down
│   │   │   │   ├── funnel.py             # Funnel analysis
│   │   │   │   ├── cohort.py             # Retention / cohort analysis
│   │   │   │   └── attribution.py        # Attribution analysis
│   │   │   │
│   │   │   ├── search/               # External intelligence
│   │   │   │   ├── industry.py           # Industry data/reports
│   │   │   │   ├── competitor.py         # Competitor tracking
│   │   │   │   └── news.py              # Industry news
│   │   │   │
│   │   │   └── output/               # Strategy output
│   │   │       ├── action_plan.py        # Executable action checklist
│   │   │       ├── ab_test.py            # A/B test design
│   │   │       └── roi_estimate.py       # ROI estimation
│   │   │
│   │   └── types.py
│   │
│   ├── pyproject.toml            # uv / poetry
│   └── .env.example
│
├── web/                          # Vite + React + ECharts — Frontend
│   ├── src/
│   │   ├── pages/
│   │   │   ├── chat/                 # Chat UI (streaming)
│   │   │   ├── dashboard/            # Dashboard + chat sidebar
│   │   │   └── reports/              # Report generator → HTML
│   │   ├── components/
│   │   │   ├── charts/               # ECharts wrapper components
│   │   │   └── ui/                   # Base UI components
│   │   ├── hooks/
│   │   ├── lib/
│   │   └── types/
│   ├── package.json
│   └── vite.config.ts
│
├── data/                         # Runtime data (gitignored)
│   ├── MEMORY.md
│   ├── settings.json
│   └── sessions/
│
├── docker-compose.yml            # Full stack deployment
└── .env.example
```

---

## Module Ownership

| Directory | Language | Responsibility |
|-----------|----------|----------------|
| `agent-server/src/` | TypeScript (Bun) | Agent Loop, MCP client, sessions, HTTP/WS |
| `agent-server/native/` | Rust | Vector search, embeddings, scheduler |
| `data-mcp/` | Python | Data query, analysis, search, strategy |
| `web/` | TypeScript (React) | Chat UI, Dashboard, Report generator |

### Cross-module Communication

```
Web ──WebSocket/REST──→ Agent Server ──MCP Protocol──→ Data MCP (Python)
                              │
                              ├──NAPI-RS──→ Rust native modules
                              │
                              └──MCP Protocol──→ Other MCP Servers (exa, etc.)
```

---

## Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| TS files | kebab-case | `agent-loop.ts`, `tool-registry.ts` |
| Rust files | snake_case | `vector_search.rs`, `lib.rs` |
| Python files | snake_case | `anomaly.py`, `drilldown.py` |
| React components | PascalCase | `ChatPanel.tsx`, `MetricCard.tsx` |
| Directories | kebab-case | `agent-server/`, `data-mcp/` |
| Classes (TS) | PascalCase | `AgentSession`, `ToolRegistry` |
| Functions (TS) | camelCase | `runAgenticLoop()` |
| Functions (Python) | snake_case | `detect_anomaly()` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_CONTEXT_TOKENS` |
| Env variables | SCREAMING_SNAKE_CASE | `LOG_LEVEL`, `LLM_PROVIDER` |
