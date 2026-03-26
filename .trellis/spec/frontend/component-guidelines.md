# Component Guidelines

> How components are built in the dashboard.

---

## Overview

Components follow standard React patterns with TypeScript strict typing. The dashboard is a monitoring/management UI — prioritize clarity and data density over visual complexity.

---

## Component Structure

```typescript
// Standard component file structure

// 1. Imports
import { useState } from 'react';
import type { Session } from '@/types';

// 2. Props interface (exported for testing)
export interface SessionCardProps {
  session: Session;
  onSelect: (id: string) => void;
  isActive?: boolean;
}

// 3. Component (named export, not default)
export function SessionCard({ session, onSelect, isActive = false }: SessionCardProps) {
  // 4. Hooks first
  const [expanded, setExpanded] = useState(false);

  // 5. Handlers
  const handleClick = () => onSelect(session.id);

  // 6. Render
  return (
    <div className={isActive ? 'card active' : 'card'} onClick={handleClick}>
      <h3>{session.id}</h3>
      <span>{session.turnCount} turns</span>
    </div>
  );
}
```

---

## Props Conventions

```typescript
// GOOD — explicit interface, exported
export interface ToolListProps {
  tools: ToolDefinition[];
  onExecute: (name: string, args: unknown) => Promise<void>;
  filter?: string;
}

// BAD — inline type, not reusable
function ToolList(props: { tools: any[]; onExecute: Function }) {}
```

**Rules:**
- Always define a named `Props` interface
- Export props interface for testing
- Use `children?: ReactNode` only when component is a container
- Default values via destructuring: `{ filter = '' }: ToolListProps`

---

## Styling Patterns

Use **CSS Modules** for component styles, or **Tailwind CSS** if configured.

```typescript
// CSS Modules
import styles from './SessionCard.module.css';

<div className={styles.card}>...</div>
```

**Rules:**
- No inline styles except for truly dynamic values (e.g., `style={{ width: `${percent}%` }}`)
- No global CSS classes in component files
- Color tokens and spacing values from design system, not hardcoded

---

## Accessibility

- All interactive elements must be keyboard-accessible
- Use semantic HTML (`<button>` not `<div onClick>`)
- Images need `alt` text
- Form inputs need associated labels

---

## Common Mistakes

1. **Default exports** — Use named exports for better IDE support and refactoring
2. **Prop drilling > 2 levels** — Extract to context or composition
3. **Fetching data in components** — Use custom hooks; components should be pure renderers
4. **Inline anonymous functions in render** — Extract as named handlers when used with `useCallback`
