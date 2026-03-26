# Quality Guidelines

> Code standards, testing requirements, and forbidden patterns.

---

## Overview

This is a polyglot project. Each language has its own quality standards:

- **TypeScript (Bun)**: strict mode, vitest, biome linter
- **Rust**: clippy zero warnings, cargo test
- **Python**: ruff linter, pytest, strict type hints

---

## TypeScript (Agent Server)

### Config

```jsonc
// tsconfig.json — strict mode required
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "exactOptionalPropertyTypes": true,
    "module": "ESNext",
    "target": "ES2022"
  }
}
```

### Before Every Commit

```bash
bun run lint        # biome check
bun run typecheck   # tsc --noEmit
bun run test        # vitest
```

### Forbidden

| Pattern | Use Instead |
|---------|-------------|
| `any` type | `unknown` + type guard |
| `console.log` | `logger.info()` |
| `!` non-null assertion | Proper null checks |
| `eval()` | Structured tool execution |
| `../../../` imports | `@/` alias or package imports |

---

## Rust (Native Modules)

### Before Every Commit

```bash
cargo clippy -- -D warnings   # Zero warnings
cargo test                     # All tests pass
cargo fmt -- --check           # Formatting
```

### Conventions

- Zero `clippy` warnings policy
- No `.unwrap()` or `.expect()` in production code
- Use `thiserror` for error types, `anyhow` for application errors
- Prefer strong types over strings (enums, newtypes)

---

## Python (Data MCP)

### Config

```toml
# pyproject.toml
[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "ANN"]

[tool.mypy]
strict = true
```

### Before Every Commit

```bash
ruff check .        # Lint
mypy .              # Type check
pytest              # Tests
```

### Conventions

- Type hints on all function signatures
- polars not pandas
- pydantic for input/output validation
- async for all I/O operations

---

## Testing

### Framework by Language

| Language | Framework | Config |
|----------|-----------|--------|
| TypeScript | vitest | `vitest.config.ts` |
| Rust | cargo test | `#[cfg(test)]` modules |
| Python | pytest | `pyproject.toml` |

### What to Test (Priority Order)

| Component | Priority | Why |
|-----------|----------|-----|
| Tool descriptions (ACI accuracy) | **Critical** | #1 cause of Agent errors |
| Tool execution (happy + error paths) | **Critical** | Agent reliability |
| Analysis correctness (metrics, anomaly) | **Critical** | Business decisions depend on it |
| Context compaction | High | Affects long conversations |
| Memory search (BM25 + vector) | High | Affects recall quality |
| Session management | High | Multi-user correctness |
| Rust native modules | High | Must not crash |
| Error recovery chain | Medium | Resilience |

### Agent Evaluation (Harness)

**Fix evaluation first, then fix the Agent.**

- **Pass@k** — Development: run k times, pass if any succeeds
- **Pass^k** — Pre-release: run k times, pass only if all succeed
- Evaluator priority: Code evaluator → Model evaluator → Human evaluator

---

## Import Order (TypeScript)

```typescript
// 1. External packages
import { z } from 'zod';

// 2. Internal aliases
import { AgentSession } from '@/agent/session';
import type { ToolDefinition } from '@/tools/registry';

// 3. Relative imports
import { validateInput } from './validator';
import type { Config } from './types';
```

Always use `import type` for type-only imports.

---

## Common Mistakes

1. **Testing happy paths only** — Tool errors and edge cases break Agents
2. **Evaluating output, not outcome** — Check what changed (DB state), not what was said
3. **Over-mocking** — Mock LLM calls, but test real DuckDB and file operations
4. **Ignoring tool description quality** — Debug selection errors by checking descriptions first
5. **Different lint configs per language** — Each language has its own; run all three before commit
