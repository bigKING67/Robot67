# Quality Guidelines

> Code quality standards for frontend development.

---

## Overview

Frontend quality follows the same principles as backend: strict TypeScript, zero warnings, comprehensive testing. The dashboard is a management tool — correctness and performance matter more than visual polish.

---

## Forbidden Patterns

| Pattern | Why | Use Instead |
|---------|-----|-------------|
| Default exports | Poor IDE refactoring support | Named exports |
| `any` type | Type safety disabled | `unknown` + narrowing |
| `console.log` | Should not ship to production | Logger or remove |
| Inline styles (non-dynamic) | Hard to maintain | CSS modules / Tailwind |
| Direct API calls in components | Mixing concerns | Custom hooks |
| `index.tsx` barrel files | Slows bundling, circular deps | Direct imports |

---

## Required Patterns

- Named exports for all components
- Props interface defined and exported
- Custom hooks for all data fetching
- Error boundaries at page level
- Loading states for async operations
- Semantic HTML elements

---

## Testing Requirements

### What to Test

| Component | Test Type | Priority |
|-----------|----------|----------|
| Data hooks | Unit (mock API) | **Critical** |
| Form validation | Unit | High |
| Page components | Integration (render + interact) | High |
| UI components | Snapshot / visual | Medium |

### Framework: vitest + React Testing Library

```typescript
import { render, screen } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { SessionCard } from './SessionCard';

describe('SessionCard', () => {
  it('should display session ID', () => {
    render(<SessionCard session={mockSession} onSelect={() => {}} />);
    expect(screen.getByText(mockSession.id)).toBeInTheDocument();
  });
});
```

---

## Code Review Checklist

- [ ] No `any` types
- [ ] No default exports
- [ ] Props interface is exported
- [ ] Data fetching is in hooks, not components
- [ ] Loading and error states handled
- [ ] Semantic HTML used
- [ ] Keyboard accessible
- [ ] No hardcoded strings (use constants)
