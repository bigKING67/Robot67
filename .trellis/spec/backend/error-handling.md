# Error Handling

> How errors are handled in this project.

---

## Overview

This is a polyglot project (TypeScript + Rust + Python). Each language handles errors with its own conventions, but all follow the same principle:

**Core principle:** Errors should be **actionable** — every error message should tell the caller what went wrong and suggest how to fix it.

- **TypeScript (Agent Server)**: Custom error classes extending `AgentError`
- **Rust (native modules)**: `thiserror` for types, `anyhow` for application errors
- **Python (Data MCP)**: Structured error dicts returned via MCP, never thrown

---

## Error Types

### Base Error Hierarchy

```typescript
// src/errors.ts

/** Base error for all agent errors */
export class AgentError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly cause?: Error
  ) {
    super(message);
    this.name = 'AgentError';
  }
}

/** Context window exceeded */
export class ContextOverflowError extends AgentError {
  constructor(
    public readonly tokenCount: number,
    public readonly maxTokens: number,
    cause?: Error
  ) {
    super(
      `Context overflow: ${tokenCount}/${maxTokens} tokens`,
      'CONTEXT_OVERFLOW',
      cause
    );
  }
}

/** Tool execution failed */
export class ToolExecutionError extends AgentError {
  constructor(
    public readonly toolName: string,
    message: string,
    cause?: Error
  ) {
    super(
      `Tool "${toolName}" failed: ${message}`,
      'TOOL_EXECUTION_FAILED',
      cause
    );
  }
}

/** LLM provider error */
export class LlmProviderError extends AgentError {
  constructor(
    public readonly provider: string,
    message: string,
    public readonly retryable: boolean = false,
    cause?: Error
  ) {
    super(
      `LLM provider "${provider}": ${message}`,
      'LLM_PROVIDER_ERROR',
      cause
    );
  }
}

/** Memory operation failed */
export class MemoryError extends AgentError {
  constructor(message: string, cause?: Error) {
    super(message, 'MEMORY_ERROR', cause);
  }
}

/** Channel communication error */
export class ChannelError extends AgentError {
  constructor(
    public readonly channel: string,
    message: string,
    cause?: Error
  ) {
    super(
      `Channel "${channel}": ${message}`,
      'CHANNEL_ERROR',
      cause
    );
  }
}
```

---

## Error Handling Patterns

### 1. Context Overflow → Compaction → Retry

Follow pi-mono's auto-recovery pattern:

```typescript
// Reference: pi-mono context overflow handling
async function runAgenticLoop(session: AgentSession): Promise<void> {
  while (true) {
    try {
      const response = await llm.chat(session.getContext());
      // ... process response
    } catch (error) {
      if (error instanceof ContextOverflowError) {
        await session.compact(); // Summarize old messages
        continue;                // Retry with compacted context
      }
      throw error; // Re-throw non-recoverable errors
    }
  }
}
```

### 2. Tool Errors → Structured Feedback to LLM

Tool errors are returned to the LLM as structured tool results, not thrown:

```typescript
// Reference: ironclaw tool error handling
async function executeTool(call: ToolCall): Promise<ToolResult> {
  try {
    const result = await registry.execute(call.name, call.args);
    return { success: true, data: result };
  } catch (error) {
    // Return error to LLM so it can self-correct
    return {
      success: false,
      error: {
        code: error instanceof AgentError ? error.code : 'UNKNOWN',
        message: error.message,
        suggestion: getSuggestion(error),
      },
    };
  }
}
```

### 3. LLM Provider Failover

```typescript
// Reference: ironclaw provider fault switching
async function chatWithFailover(messages: Message[]): Promise<Response> {
  const providers = registry.getProviders(); // Ordered by priority

  for (const provider of providers) {
    try {
      return await provider.chat(messages);
    } catch (error) {
      if (error instanceof LlmProviderError && error.retryable) {
        logger.warn({ provider: provider.name, error }, 'Provider failed, trying next');
        continue;
      }
      throw error;
    }
  }

  throw new LlmProviderError('all', 'All providers exhausted', false);
}
```

### 4. Boundary Error Handling (API/Channel)

At system boundaries (HTTP endpoints, MCP handlers), return structured responses:

```typescript
// Reference: universal-db-mcp structured error response
interface ErrorResponse {
  success: false;
  error: {
    code: string;      // Machine-readable: 'TOOL_EXECUTION_FAILED'
    message: string;   // Human-readable description
  };
  metadata?: {
    requestId: string;
  };
}
```

---

## API Error Responses

### Standard Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `CONTEXT_OVERFLOW` | - | Context too large, trigger compaction |
| `TOOL_EXECUTION_FAILED` | - | Tool returned an error |
| `TOOL_NOT_FOUND` | 400 | Requested tool does not exist |
| `LLM_PROVIDER_ERROR` | 502 | LLM API call failed |
| `MEMORY_ERROR` | 500 | Memory read/write failed |
| `CHANNEL_ERROR` | 500 | Channel I/O error |
| `VALIDATION_ERROR` | 400 | Input failed schema validation |
| `AUTH_ERROR` | 401 | Authentication failed |
| `RATE_LIMITED` | 429 | Too many requests |

---

## Common Mistakes

1. **Swallowing errors silently** — Always log or propagate; never `catch (e) {}`
2. **Throwing inside tool execution** — Tool errors should be returned as ToolResult, not thrown
3. **Not including `cause`** — Always chain the original error for debugging
4. **Generic error messages** — Include context: which tool, which provider, what input
5. **Using `.unwrap()` or non-null assertions** — Use proper null checks and error types
6. **Retrying non-retryable errors** — Check `retryable` flag before retry loops

---

## Forbidden Patterns

```typescript
// BAD: Silent catch
try { await tool.execute(); } catch (e) { /* nothing */ }

// BAD: Losing error context
try { await tool.execute(); } catch (e) { throw new Error('Tool failed'); }

// GOOD: Preserve cause chain
try {
  await tool.execute();
} catch (e) {
  throw new ToolExecutionError(tool.name, 'Execution failed', e as Error);
}
```
