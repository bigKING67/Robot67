# Hook Guidelines

> How hooks are used in the dashboard.

---

## Overview

Custom hooks encapsulate data fetching, WebSocket connections, and shared stateful logic. All data access goes through hooks — components should not call APIs directly.

---

## Custom Hook Patterns

```typescript
// hooks/use-agent-api.ts

import { useState, useEffect, useCallback } from 'react';
import { apiClient } from '@/lib/api-client';
import type { Session } from '@/types';

interface UseSessionsResult {
  sessions: Session[];
  loading: boolean;
  error: Error | null;
  refresh: () => void;
}

export function useSessions(): UseSessionsResult {
  const [sessions, setSessions] = useState<Session[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchSessions = useCallback(async () => {
    setLoading(true);
    try {
      const data = await apiClient.getSessions();
      setSessions(data);
      setError(null);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchSessions();
  }, [fetchSessions]);

  return { sessions, loading, error, refresh: fetchSessions };
}
```

---

## Data Fetching

Use custom hooks wrapping `fetch` or a lightweight client. If the project grows, consider `@tanstack/react-query` for caching and deduplication.

**Rules:**
- Every API endpoint gets its own hook
- Hooks return `{ data, loading, error, refresh }` consistently
- Error state is typed as `Error | null`, not `unknown`

---

## Naming Conventions

| Pattern | Example |
|---------|---------|
| Data fetching | `useSessions()`, `useMemorySearch(query)` |
| WebSocket | `useAgentStream(sessionId)` |
| UI state | `useSidebar()`, `useTheme()` |
| Form logic | `useToolForm()` |

Prefix: always `use`. File name matches hook name: `use-sessions.ts`.

---

## Common Mistakes

1. **Fetching in components** — Always use hooks; keeps components pure
2. **Missing cleanup** — WebSocket and subscription hooks must return cleanup in `useEffect`
3. **Stale closures** — Use `useCallback` for functions passed as props
4. **Infinite re-render loops** — Object/array dependencies in `useEffect` need `useMemo`
