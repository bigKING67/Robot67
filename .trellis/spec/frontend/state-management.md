# State Management

> How state is managed in the dashboard.

---

## Overview

The dashboard is a relatively simple management UI. State management follows the principle of **least complexity** — use the simplest tool that works.

---

## State Categories

| Category | Tool | Example |
|----------|------|---------|
| **UI State** | `useState` / `useReducer` | Sidebar open, active tab, filter text |
| **Server State** | Custom hooks (or React Query) | Sessions, memories, tool list |
| **Shared State** | React Context | Current session, theme, user settings |
| **URL State** | URL params / search params | Selected session ID, search query |

---

## When to Use Global State

Use React Context only when:
- Data is needed by 3+ unrelated components
- Data changes infrequently (theme, auth, settings)
- Prop drilling would cross 3+ levels

**Do NOT use global state for:**
- Server data (use fetching hooks with caching)
- Form state (keep local to form component)
- Transient UI state (keep in the owning component)

---

## Server State

Server data is the primary state in this dashboard. Manage via custom hooks:

```typescript
// Pattern: hook per resource
const { sessions, loading, refresh } = useSessions();
const { memories, search } = useMemorySearch(query);
const { traces } = useTraces(sessionId);
```

If data needs caching/deduplication across components, introduce React Query:

```typescript
import { useQuery } from '@tanstack/react-query';

export function useSessions() {
  return useQuery({
    queryKey: ['sessions'],
    queryFn: () => apiClient.getSessions(),
    staleTime: 5000,
  });
}
```

---

## Common Mistakes

1. **Premature global state** — Start with `useState`, escalate only when needed
2. **Storing server data in global state** — Use fetching hooks; server is the source of truth
3. **Context for frequently changing values** — Causes unnecessary re-renders; use `useMemo`
4. **Derived state as stored state** — Compute it; don't store what you can derive
