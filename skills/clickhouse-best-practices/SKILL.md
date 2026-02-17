---
name: clickhouse-best-practices
description: Comprehensive ClickHouse best practices for schema design, query optimization, and data ingestion. Use when user says "create a ClickHouse table", "optimize this query", "why is this query slow", "design a schema for", or works with CREATE TABLE, ORDER BY, JOINs, or INSERT patterns. Covers 28 rules across primary keys, data types, partitioning, JOINs, and mutations. Do NOT use for general SQL databases (PostgreSQL, MySQL, SQLite) — this is specific to ClickHouse MergeTree engine family.
license: Apache-2.0
compatibility: Requires ClickHouse 24.1+. Works with open-source ClickHouse and ClickHouse Cloud.
metadata:
  author: ClearTax
  version: "0.4.0"
---

# ClickHouse Best Practices

28 rules covering schema design, query optimization, and data ingestion for ClickHouse — prioritized by impact.

> **Official docs:** [ClickHouse Best Practices](https://clickhouse.com/docs/best-practices)

## IMPORTANT: How to Apply This Skill

**Before answering ClickHouse questions, follow this priority order:**

1. **Check for applicable rules** in the `rules/` directory
2. **If rules exist:** Apply them and cite them in your response using "Per `rule-name`..."
3. **If no rule exists:** Use ClickHouse knowledge or search documentation
4. **If uncertain:** Use web search for current best practices
5. **Always cite your source:** rule name, "general ClickHouse guidance", or URL

**Why rules take priority:** ClickHouse has specific behaviors (columnar storage, sparse indexes, merge tree mechanics) where general database intuition can be misleading. The rules encode validated, ClickHouse-specific guidance.

## Rule Categories by Priority

| Priority | Category | Impact | Prefix | Rules |
|----------|----------|--------|--------|-------|
| 1 | Primary Key Selection | CRITICAL | `schema-pk-` | 4 |
| 2 | Data Type Selection | CRITICAL | `schema-types-` | 5 |
| 3 | JOIN Optimization | CRITICAL | `query-join-` | 5 |
| 4 | Insert Batching | CRITICAL | `insert-batch-` | 1 |
| 5 | Mutation Avoidance | CRITICAL | `insert-mutation-` | 2 |
| 6 | Partitioning Strategy | HIGH | `schema-partition-` | 4 |
| 7 | Skipping Indices | HIGH | `query-index-` | 1 |
| 8 | Materialized Views | HIGH | `query-mv-` | 2 |
| 9 | Async Inserts | HIGH | `insert-async-` | 2 |
| 10 | OPTIMIZE Avoidance | HIGH | `insert-optimize-` | 1 |
| 11 | JSON Usage | MEDIUM | `schema-json-` | 1 |

## When to Apply

This skill activates when you encounter:

- `CREATE TABLE` or `ALTER TABLE` statements
- `ORDER BY` or `PRIMARY KEY` discussions
- Data type selection questions
- Slow query troubleshooting
- JOIN optimization requests
- Data ingestion pipeline design
- Update/delete strategy questions
- ReplacingMergeTree or other specialized engine usage
- Partitioning strategy decisions

**Do NOT activate for:** General SQL questions targeting PostgreSQL, MySQL, SQLite, or other non-ClickHouse databases.

## Examples

### Example 1: Create a table for user events

User says: "Create a table for storing user events with user_id, event_type, properties, and timestamp"

1. Read `rules/schema-pk-plan-before-creation.md` — plan ORDER BY based on query patterns
2. Read `rules/schema-types-native-types.md` — use DateTime not String for timestamp
3. Read `rules/schema-types-lowcardinality.md` — apply LowCardinality to event_type
4. Read `rules/schema-partition-start-without.md` — consider if partitioning is needed
5. Apply rules and cite them: "Per `schema-pk-cardinality-order`, columns are ordered low-to-high cardinality"

Result: Table with query-driven primary key, proper types, bounded partitions.

### Example 2: Optimize a slow JOIN query

User says: "This JOIN query is taking too long"

1. Read `rules/query-join-filter-before.md` — check if tables are filtered before joining
2. Read `rules/query-join-choose-algorithm.md` — verify correct algorithm for table sizes
3. Read `rules/query-join-use-any.md` — check if ANY JOIN applies
4. Read `rules/schema-pk-filter-on-orderby.md` — verify filters align with ORDER BY

Result: Optimized query with pre-filtered tables and correct JOIN algorithm.

## For Formal Reviews

When performing a formal review, consult `references/review-procedures.md` for:
- Schema review checklist (CREATE TABLE, ALTER TABLE)
- Query review checklist (SELECT, JOIN, aggregations)
- Insert strategy review checklist (data ingestion, updates, deletes)

For structured output formatting, consult `references/output-format.md`.

For the full rule listing by category, consult `references/quick-reference.md`.

## Common Mistakes to Watch For

- **String overuse**: Using `String` for dates, numbers, or low-cardinality values instead of native types
- **Wrong ORDER BY**: Choosing ORDER BY columns without analyzing query patterns — this is immutable
- **Too many partitions**: Partition key with >1,000 values causes excessive parts and slow queries
- **Post-JOIN filtering**: Filtering after JOIN instead of before, scanning unnecessary data
- **Using ALTER UPDATE**: Using mutations for frequent updates instead of ReplacingMergeTree
- **Small INSERT batches**: Inserting <10K rows per batch without async inserts enabled

## Performance Notes

- Take your time to check all applicable rules thoroughly
- Quality is more important than speed — a wrong ORDER BY is permanent
- Do not skip validation checklists for schema reviews

## Rule File Structure

Each rule file in `rules/` contains:

- **YAML frontmatter**: title, impact level, tags
- **Brief explanation**: Why this rule matters
- **Incorrect example**: Anti-pattern with explanation
- **Correct example**: Best practice with explanation
- **Additional context**: Trade-offs, when to apply, references

## Full Compiled Document

For the complete guide with all rules expanded inline: `AGENTS.md`

Use `AGENTS.md` when you need to check multiple rules quickly without reading individual files.
