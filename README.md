# ClickHouse Agent Skills

Agent Skills that help LLMs and agents adopt best practices when working with ClickHouse. These skills cover schema design, query optimization, data ingestion patterns, and driver/client library development.

When an agent loads these skills, it gains knowledge of ClickHouse best practices and can apply them while helping you design tables, write queries, troubleshoot performance issues, or build ClickHouse integrations.

## Available Skills

### ClickHouse Best Practices

**28 rules** covering schema design, query optimization, and data ingestion—prioritized by impact.

| Category | Rules | Impact |
|----------|-------|--------|
| Primary Key Selection | 4 | CRITICAL |
| Data Type Selection | 5 | CRITICAL |
| JOIN Optimization | 5 | CRITICAL |
| Insert Batching | 1 | CRITICAL |
| Mutation Avoidance | 2 | CRITICAL |
| Partitioning Strategy | 4 | HIGH |
| Skipping Indices | 1 | HIGH |
| Materialized Views | 2 | HIGH |
| Async Inserts | 2 | HIGH |
| OPTIMIZE Avoidance | 1 | HIGH |
| JSON Usage | 1 | MEDIUM |

**28 atomic rules** organized by prefix:

| Prefix | Count | Coverage |
|--------|-------|----------|
| `schema-pk-*` | 4 | PRIMARY KEY selection, cardinality ordering |
| `schema-types-*` | 5 | Data types, LowCardinality, Nullable |
| `schema-partition-*` | 4 | Partitioning strategy, lifecycle management |
| `schema-json-*` | 1 | JSON type usage |
| `query-join-*` | 5 | JOIN algorithms, filtering, alternatives |
| `query-index-*` | 1 | Data skipping indices |
| `query-mv-*` | 2 | Incremental and refreshable MVs |
| `insert-batch-*` | 1 | Batch sizing (10K-100K rows) |
| `insert-async-*` | 2 | Async inserts, data formats |
| `insert-mutation-*` | 2 | Mutation avoidance |
| `insert-optimize-*` | 1 | OPTIMIZE FINAL avoidance |

**Trigger phrases** — the skill activates when you say:
- "Create a table for..."
- "Optimize this query..."
- "Design a schema for..."
- "Why is this query slow?"
- "How should I insert data into..."
- "Should I use UPDATE or..."

**Location:** [`skills/clickhouse-best-practices/`](./skills/clickhouse-best-practices/)

**For humans:** Read [SKILL.md](./skills/clickhouse-best-practices/SKILL.md) for an overview, or [AGENTS.md](./skills/clickhouse-best-practices/AGENTS.md) for the complete compiled guide.

**For agents:** The skill activates automatically when you work with ClickHouse—creating tables, writing queries, or designing data pipelines.

### ClickHouse Driver Best Practices

**12 rules** for building reliable ClickHouse client libraries and integrations.

| Category | Rules | Impact |
|----------|-------|--------|
| Retry & Resilience | 3 | CRITICAL |
| Connection Management | 3 | CRITICAL |
| Query Execution | 3 | HIGH |
| Observability | 3 | HIGH |

**12 rules** organized by prefix:

| Prefix | Count | Coverage |
|--------|-------|----------|
| `retry-*` | 3 | Transient error retry, transparent proxy, structured logging |
| `conn-*` | 3 | Annotation-driven activation, optional injection, timeouts |
| `query-*` | 3 | Mutation IDs, timeout polling, format selection |
| `obs-*` | 3 | Query timing, response cleanup, named parameters |

**Trigger phrases** — the skill activates when you say:
- "Build a ClickHouse client..."
- "Configure ClickHouse connection..."
- "Add retry to ClickHouse..."
- "Handle ClickHouse timeouts..."
- "Write a ClickHouse driver..."

**Location:** [`skills/clickhouse-driver-best-practices/`](./skills/clickhouse-driver-best-practices/)

**For humans:** Read [SKILL.md](./skills/clickhouse-driver-best-practices/SKILL.md) for an overview.

**For agents:** The skill activates automatically when you work with ClickHouse client/driver code.

---

## Quick Start

After installation, your AI agent will reference these best practices when:

- Creating new tables with `CREATE TABLE`
- Choosing `ORDER BY` / `PRIMARY KEY` columns
- Selecting data types for columns
- Optimizing slow queries
- Writing or tuning JOINs
- Designing data ingestion pipelines
- Handling updates or deletes

Example prompt:
> "Create a table for storing user events with fields for user_id, event_type, properties (JSON), and timestamp"

The agent will apply relevant rules like proper column ordering in the primary key, appropriate data types, and partitioning strategy.

## Supported Agents

Skills are **agent-agnostic**—the same skill works across all supported AI coding assistants:

| Agent | Config Directory |
|-------|------------------|
| [Claude Code](https://claude.ai/code) | `.claude/skills/` |
| [Cursor](https://cursor.sh) | `.cursor/skills/` |
| [Windsurf](https://codeium.com/windsurf) | `.windsurf/skills/` |
| [GitHub Copilot](https://github.com/features/copilot) | `.github/skills/` |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | `.gemini/skills/` |
| [Cline](https://github.com/cline/cline) | `.cline/skills/` |
| [Codex](https://openai.com/codex) | `.codex/skills/` |
| [Goose](https://github.com/block/goose) | `.goose/skills/` |
| [Roo Code](https://roo.ai) | `.roo/skills/` |
| [OpenHands](https://github.com/All-Hands-AI/OpenHands) | `.openhands/skills/` |

And 13 more including Amp, Kiro CLI, Trae, Zencoder, and others.

The installer detects which agents you have by checking for their configuration directories. If an agent isn't listed, either install it first or create its config directory manually (e.g., `mkdir -p ~/.cursor`).

## License

Apache 2.0 — see [LICENSE](./LICENSE) for details.
