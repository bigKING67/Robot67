# Frontend Development Guidelines

> Best practices for frontend development in this project.

---

## Overview

The frontend is a **Vite + React SPA** providing three interaction modes for the data analysis Agent:

1. **Chat UI** — Conversational interface with streaming responses
2. **Dashboard + Chat** — Fixed metric cards with chat sidebar
3. **Report Generator** — Template-based HTML reports with ECharts

No SSR framework (no Next.js/Remix) — pure client-side SPA. Dashboard doesn't need SEO.

---

## Guidelines Index

| Guide | Description | Status |
|-------|-------------|--------|
| [Directory Structure](./directory-structure.md) | SPA layout, pages, components | Filled |
| [Component Guidelines](./component-guidelines.md) | Component patterns, props, composition | Filled |
| [Hook Guidelines](./hook-guidelines.md) | Custom hooks, data fetching, WebSocket | Filled |
| [State Management](./state-management.md) | Local state, server state, context | Filled |
| [Quality Guidelines](./quality-guidelines.md) | Code standards, testing, forbidden patterns | Filled |
| [Type Safety](./type-safety.md) | Zod validation, type organization | Filled |

---

## Pre-Development Checklist

1. **Always**: [Directory Structure](./directory-structure.md) — Know where files go
2. **Always**: [Component Guidelines](./component-guidelines.md) — Named exports, props patterns
3. **If fetching data**: [Hook Guidelines](./hook-guidelines.md) — Custom hooks for API
4. **If sharing state**: [State Management](./state-management.md) — Least complexity
5. **Before committing**: [Quality Guidelines](./quality-guidelines.md) — Lint, test, review

---

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Build tool | **Vite** | Fast HMR, no SSR overhead |
| UI framework | **React** | Largest ecosystem |
| Charts | **ECharts** | Best data visualization, interactive |
| Styling | Tailwind CSS | Utility-first, consistent |
| State | useState + hooks → React Query | Start simple, escalate |
| Validation | zod | Shared with backend |
| Testing | vitest + React Testing Library | Consistent with Agent Server |

---

## Reference Projects

| Project | URL | Frontend Reference |
|---------|-----|--------------------|
| **cortex** | https://github.com/rikouu/cortex | `packages/dashboard/` — React SPA for memory management |
| **pi-mono** | https://github.com/badlogic/pi-mono | `packages/web-ui/` — Browser-based chat interface |
| **ironclaw** | https://github.com/nearai/ironclaw | Web gateway — HTTP channel, session UI patterns |
| **universal-db-mcp** | https://github.com/Anarkh-Lee/universal-db-mcp | REST API design — reference for API client |

---

**Language**: All documentation should be written in **English**.
