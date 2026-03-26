# Database Guidelines

> Storage patterns, queries, migrations, and persistence conventions.

---

## Overview

This project uses a **multi-backend storage** strategy:

- **PostgreSQL** — Business data source (user's existing DB)
- **DuckDB** — OLAP analysis engine for CSV/Parquet/cross-source queries
- **SQLite** — Memory store, metadata, search indexes (WAL mode)
- **JSONL files** — Session history, conversation logs (append-only)
- **Markdown files** — Long-term semantic memory (MEMORY.md)

Reference patterns: pi-mono (JSONL sessions), cortex (SQLite + FTS5 + vector), ironclaw (migrations), universal-db-mcp (adapter pattern).

---

## Storage Patterns

### 1. JSONL for Session History

Session data uses append-only JSONL files with conversation branching support.

```typescript
// Reference: pi-mono log.jsonl format
interface SessionEntry {
  id: string;            // UUID
  parentId?: string;     // For conversation branching
  role: 'user' | 'assistant' | 'system' | 'tool';
  content: string;
  timestamp: string;     // ISO 8601
  metadata?: Record<string, unknown>;
}
```

**Rules:**
- `log.jsonl` — Permanent, never truncated (full history)
- `context.jsonl` — LLM context, subject to compaction
- One line per entry, newline-delimited
- Always append, never overwrite existing lines

### 2. SQLite for Memory Store

Memory uses SQLite with WAL mode for concurrent read/write access.

```typescript
// Reference: cortex database initialization
function initDatabase(dbPath: string): Database {
  const dir = path.dirname(dbPath);
  fs.mkdirSync(dir, { recursive: true });

  const db = new Database(dbPath);
  db.pragma('journal_mode = WAL');
  db.pragma('foreign_keys = ON');

  runMigrations(db);
  return db;
}
```

**Rules:**
- Always enable WAL mode: `PRAGMA journal_mode = WAL`
- Always enable foreign keys: `PRAGMA foreign_keys = ON`
- Use `better-sqlite3` (synchronous, faster for single-process)
- Use FTS5 virtual tables for full-text search
- Use numbered migration files: `001_initial_schema.ts`, `002_add_vectors.ts`

### 3. Memory Layers

Follow cortex's three-layer memory architecture:

| Layer | Purpose | TTL | Max entries |
|-------|---------|-----|-------------|
| `working` | Short-term, session-scoped | 48 hours | Unlimited |
| `core` | Important, frequently accessed | Permanent | 1000 |
| `archive` | Compressed older memories | 90 days | Unlimited |

```sql
-- Reference: cortex memories table
CREATE TABLE memories (
  id TEXT PRIMARY KEY,
  layer TEXT NOT NULL CHECK (layer IN ('working', 'core', 'archive')),
  category TEXT NOT NULL,
  content TEXT NOT NULL,
  agent_id TEXT NOT NULL,
  importance REAL DEFAULT 0.5,
  confidence REAL DEFAULT 1.0,
  decay_score REAL DEFAULT 1.0,
  access_count INTEGER DEFAULT 0,
  expires_at TEXT,
  superseded_by TEXT,
  metadata TEXT DEFAULT '{}',
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now'))
);

-- FTS5 for full-text search
CREATE VIRTUAL TABLE memories_fts USING fts5(
  content, category, content=memories, content_rowid=rowid
);
```

### 4. MEMORY.md for Semantic Memory

Long-term, human-readable memory persisted as Markdown.

**Rules:**
- Keep under 200 lines (same as Claude Code convention)
- Update after significant interactions
- Global `MEMORY.md` + per-channel `MEMORY.md`

---

## Migration Patterns

### File Naming

```
migrations/
├── 001_initial_schema.ts
├── 002_add_memory_vectors.ts
├── 003_add_relations_table.ts
└── 004_add_session_index.ts
```

### Migration Template

```typescript
import type { Database } from 'better-sqlite3';

export function up(db: Database): void {
  db.exec(`
    ALTER TABLE memories ADD COLUMN embedding BLOB;
    CREATE INDEX idx_memories_layer ON memories(layer);
    CREATE INDEX idx_memories_agent ON memories(agent_id);
  `);
}

export function down(db: Database): void {
  db.exec(`
    DROP INDEX IF EXISTS idx_memories_agent;
    DROP INDEX IF EXISTS idx_memories_layer;
  `);
}
```

---

## Query Patterns

### Hybrid Search (BM25 + Vector)

Follow cortex's Reciprocal Rank Fusion pattern:

```typescript
async function hybridSearch(query: string, agentId: string): Promise<Memory[]> {
  // 1. BM25 keyword search via FTS5
  const bm25Results = db.prepare(`
    SELECT m.*, rank
    FROM memories_fts fts
    JOIN memories m ON m.rowid = fts.rowid
    WHERE memories_fts MATCH ?
    AND m.agent_id = ?
    ORDER BY rank
    LIMIT 20
  `).all(query, agentId);

  // 2. Vector similarity search
  const vectorResults = await vectorSearch(query, agentId, 20);

  // 3. Reciprocal Rank Fusion
  return fuseResults(bm25Results, vectorResults, { k: 60 });
}
```

### Always Parameterize Queries

```typescript
// GOOD
db.prepare('SELECT * FROM memories WHERE agent_id = ?').all(agentId);

// BAD — SQL injection risk
db.prepare(`SELECT * FROM memories WHERE agent_id = '${agentId}'`).all();
```

---

## Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Tables | snake_case, plural | `memories`, `relations`, `sessions` |
| Columns | snake_case | `agent_id`, `created_at`, `decay_score` |
| Indexes | `idx_{table}_{column}` | `idx_memories_agent`, `idx_memories_layer` |
| FTS tables | `{table}_fts` | `memories_fts` |
| Migration files | `{NNN}_{description}.ts` | `001_initial_schema.ts` |

---

## Forbidden Patterns

| Pattern | Why | Use Instead |
|---------|-----|-------------|
| String interpolation in SQL | SQL injection risk | Parameterized queries |
| Overwriting JSONL files | Loses history | Append-only |
| Storing secrets in SQLite | Security risk | Environment variables |
| Synchronous file writes for logs | Blocks event loop | Async/buffered writes |
| Raw `fs.writeFileSync` for JSONL | No atomicity | Write to temp + rename |

---

## Common Mistakes

1. **Forgetting WAL mode** — SQLite defaults to rollback journal, blocking concurrent reads during writes
2. **Not indexing agent_id** — Every query filters by agent_id; missing index causes full table scans
3. **Unbounded memory queries** — Always LIMIT results; use pagination for large datasets
4. **Mixing log.jsonl and context.jsonl** — Log is permanent history; context is compactable LLM state
