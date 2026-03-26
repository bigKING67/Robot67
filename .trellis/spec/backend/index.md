# Backend Development Guidelines

> Best practices for backend development in this data analysis Agent project.

---

## Overview

This is a **business-growth data analysis Agent** built with a polyglot architecture:

- **Agent Server**: TypeScript (Bun runtime) — Agent Loop, MCP Client, Session management
- **Performance Modules**: Rust (NAPI-RS or standalone service) — vector search, concurrency, compute-intensive tasks
- **Data Analysis MCP**: Python (polars + scipy) — data querying, statistical analysis, visualization data
- **Frontend**: Vite + React + ECharts — Chat UI, Dashboard, Report generator

---

## Guidelines Index

| Guide | Description | Status |
|-------|-------------|--------|
| [Directory Structure](./directory-structure.md) | Polyglot monorepo layout | Filled |
| [Database Guidelines](./database-guidelines.md) | PostgreSQL + DuckDB + JSONL + MEMORY.md | Filled |
| [Error Handling](./error-handling.md) | Typed errors, overflow recovery, tool errors | Filled |
| [Quality Guidelines](./quality-guidelines.md) | Strict TS, Rust clippy, vitest, harness eval | Filled |
| [Logging Guidelines](./logging-guidelines.md) | pino structured logging, trace events | Filled |
| [Tool Design Guidelines](./tool-design-guidelines.md) | ACI principles, data-specific tool patterns | Filled |
| [Data Engine Guidelines](./data-engine-guidelines.md) | Python MCP, polars, analysis patterns | Filled |

---

## Pre-Development Checklist

Before writing any backend code, read at minimum:

1. **Always**: [Directory Structure](./directory-structure.md) — Know where your code goes
2. **Always**: [Error Handling](./error-handling.md) — Error type hierarchy
3. **If adding tools**: [Tool Design Guidelines](./tool-design-guidelines.md) — ACI principles
4. **If touching data analysis**: [Data Engine Guidelines](./data-engine-guidelines.md) — Python MCP patterns
5. **If touching storage**: [Database Guidelines](./database-guidelines.md) — PG vs DuckDB vs JSONL
6. **Before committing**: [Quality Guidelines](./quality-guidelines.md) — Lint, type check, test

---

## Key Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Agent Server runtime | **Bun** (not Node.js) | 3-5x faster, TS native, fast iteration for Agent tuning |
| Agent Server language | **TypeScript (strict)** | Fast iteration for prompt/tool tuning, MCP SDK best support |
| Performance modules | **Rust** (NAPI-RS) | Vector search, embedding compute, concurrency scheduling |
| Data analysis | **Python** (polars + scipy) | Data ecosystem unmatched, polars is Rust-backed |
| Business DB | **PostgreSQL** | User's existing data source |
| Analysis engine | **DuckDB** | Columnar OLAP, 100-1000x faster than SQLite for analytics |
| Session storage | **JSONL** (append-only) | Branching support, compaction |
| Memory | 3-layer (working/core/archive) | Cortex pattern, lifecycle management |
| Search | BM25 + Vector (RRF fusion) | Hybrid recall for memory search |
| Logging | **pino** (structured JSON) | Fast, structured, configurable |
| Testing | **vitest** | Fast, TypeScript-native |
| Validation | **zod** | Runtime type safety at boundaries |
| LLM abstraction | Multi-provider registry | Failover, model selection |
| MCP protocol | Standard interface | Decouples Agent from tools, polyglot support |

---

## Reference Projects

These open-source projects are the architectural foundation. Track them for new features and patterns.

| Project | URL | Key Contributions |
|---------|-----|-------------------|
| **pi-mono** | https://github.com/badlogic/pi-mono | Agent Loop event model, JSONL sessions, Skills system, context compaction, branch summarization |
| **ironclaw** | https://github.com/nearai/ironclaw | Rust ToolRegistry, WASM/Docker sandbox, typed errors, workspace memory with hybrid search |
| **cortex** | https://github.com/rikouu/cortex | 3-layer memory, BM25+Vector hybrid search with RRF, memory lifecycle engine, recall pipeline |
| **universal-db-mcp** | https://github.com/Anarkh-Lee/universal-db-mcp | MCP server adapter pattern, DbAdapter interface, structured error responses, zod validation |

### What to Watch

- **pi-mono**: New tool types, compaction strategies, Skills format changes
- **ironclaw**: Rust performance patterns, NAPI-RS integration examples, security sandbox evolution
- **cortex**: Memory layer tuning, recall pipeline optimizations, new search algorithms
- **universal-db-mcp**: New database adapters, MCP protocol updates

---

**Language**: All documentation should be written in **English**.
