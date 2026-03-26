# Logging Guidelines

> Structured logging, log levels, and observability.

---

## Overview

Logging conventions per language:

- **TypeScript (Agent Server)**: **pino** for structured JSON logging (pi-mono pattern)
- **Rust (native modules)**: **tracing** + tracing-subscriber (ironclaw pattern)
- **Python (Data MCP)**: **structlog** or Python `logging` with JSON formatter

Agent execution trace events follow ironclaw's event-based observability model.

---

## Log Library

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.LOG_PRETTY === 'true'
    ? { target: 'pino-pretty' }
    : undefined,
});
```

**Environment variables:**
- `LOG_LEVEL` — `debug`, `info`, `warn`, `error` (default: `info`)
- `LOG_PRETTY` — `true` for human-readable output in development

---

## Log Levels

| Level | When to Use | Example |
|-------|-------------|---------|
| `debug` | Detailed execution info for development | Tool call arguments, LLM prompt content |
| `info` | Normal operations, milestones | Session started, tool executed, compaction triggered |
| `warn` | Recoverable issues, degraded behavior | Provider failover, rate limit approaching, retry |
| `error` | Failures requiring attention | Tool crash, LLM API failure, memory corruption |

---

## Structured Log Format

Always include contextual fields:

```typescript
// GOOD — structured with context
logger.info({ sessionId, toolName, durationMs }, 'Tool executed');
logger.error({ sessionId, provider, error: err.message }, 'LLM call failed');

// BAD — unstructured string concatenation
logger.info(`Tool ${toolName} executed in session ${sessionId}`);
```

### Required Fields by Context

| Context | Required Fields |
|---------|----------------|
| Agent loop | `sessionId`, `turnNumber` |
| Tool execution | `sessionId`, `toolName`, `durationMs` |
| LLM call | `sessionId`, `provider`, `model`, `tokenCount` |
| Memory operation | `agentId`, `operation`, `layer` |
| Channel I/O | `channelType`, `messageId` |
| Error | `error` (message), `code`, `stack` (debug only) |

---

## Agent Trace Events

Follow ironclaw's event model for full observability:

```typescript
type TraceEvent =
  | { type: 'agent_start'; sessionId: string; timestamp: string }
  | { type: 'turn_start'; sessionId: string; turnNumber: number }
  | { type: 'llm_call_start'; provider: string; model: string }
  | { type: 'llm_call_end'; tokenCount: number; durationMs: number }
  | { type: 'tool_start'; toolName: string; args: unknown }
  | { type: 'tool_end'; toolName: string; success: boolean; durationMs: number }
  | { type: 'compaction'; removedMessages: number; summaryTokens: number }
  | { type: 'turn_end'; turnNumber: number }
  | { type: 'agent_end'; totalTokens: number; totalCost: number };
```

### Trace Logging

```typescript
function logTraceEvent(event: TraceEvent): void {
  // 1. Append to trace JSONL (offline analysis)
  appendToFile(traceFile, JSON.stringify(event));

  // 2. Log for real-time monitoring
  logger.info({ trace: event.type, ...event }, `Trace: ${event.type}`);
}
```

---

## Usage Tracking

Track token usage and cost per session:

```typescript
interface UsageSummary {
  sessionId: string;
  totalInputTokens: number;
  totalOutputTokens: number;
  totalCost: number;
  turnCount: number;
  toolCallCount: number;
  durationMs: number;
}

function logUsageSummary(usage: UsageSummary): void {
  logger.info({
    sessionId: usage.sessionId,
    tokens: { input: usage.totalInputTokens, output: usage.totalOutputTokens },
    cost: `$${usage.totalCost.toFixed(4)}`,
    turns: usage.turnCount,
    tools: usage.toolCallCount,
    duration: `${(usage.durationMs / 1000).toFixed(1)}s`,
  }, 'Session complete');
}
```

---

## What NOT to Log

| Do NOT log | Why |
|-----------|-----|
| API keys, secrets | Security |
| Full LLM prompts at `info` level | Too verbose, use `debug` |
| User PII | Privacy compliance |
| Full tool output | Can be enormous; log summary + write to file |
| Binary data / embeddings | Not human-readable; log dimensions only |

---

## Common Mistakes

1. **Console.log in production** — Always use the logger; `console.log` bypasses structured logging
2. **Logging inside hot loops** — Log at loop boundaries, not per-iteration
3. **Missing sessionId** — Every log inside an agent loop must include sessionId
4. **Logging full tool output** — Write to file, log summary with byte count
5. **Not logging tool failures** — Even when returned to LLM, log at `warn` for observability
