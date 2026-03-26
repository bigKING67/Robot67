# Type Safety

> TypeScript patterns and validation in the dashboard.

---

## Overview

The dashboard shares type definitions with backend packages via the monorepo workspace. Runtime validation uses `zod` at system boundaries (API responses, user input).

---

## Type Organization

```
packages/dashboard/src/types/
├── index.ts          # Re-exports all types
├── session.ts        # Session-related types
├── memory.ts         # Memory-related types
└── tools.ts          # Tool-related types
```

**Rules:**
- Shared types between frontend and backend go in a `@dataagent/shared` package
- Dashboard-only types stay in `packages/dashboard/src/types/`
- Always use `import type` for type-only imports

---

## Validation

Validate API responses at the boundary:

```typescript
import { z } from 'zod';

// Define schema
const SessionSchema = z.object({
  id: z.string(),
  turnCount: z.number(),
  tokenCount: z.number(),
  status: z.enum(['active', 'completed', 'error']),
  createdAt: z.string().datetime(),
});

export type Session = z.infer<typeof SessionSchema>;

// Validate at API boundary
async function getSessions(): Promise<Session[]> {
  const response = await fetch('/api/sessions');
  const data = await response.json();
  return z.array(SessionSchema).parse(data);
}
```

---

## Common Patterns

### Type Guards

```typescript
function isErrorResponse(data: unknown): data is { error: { code: string; message: string } } {
  return (
    typeof data === 'object' && data !== null &&
    'error' in data && typeof (data as any).error?.code === 'string'
  );
}
```

### Discriminated Unions for State

```typescript
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

---

## Forbidden Patterns

| Pattern | Why | Use Instead |
|---------|-----|-------------|
| `any` | Disables type safety | `unknown` + type guard |
| `as` type assertion | Hides bugs | Type narrowing or `zod.parse()` |
| `@ts-ignore` | Silences real errors | Fix the type error |
| Untyped API responses | Runtime crashes | Zod validation at boundary |
| `interface` for props | Less composable | `type` for component props, `interface` for classes |
