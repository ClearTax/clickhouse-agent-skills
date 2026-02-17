---
name: clickhouse-driver-best-practices
description: Best practices for building ClickHouse client libraries, drivers, and Spring Boot integrations. Use when user says "build a ClickHouse client", "write a ClickHouse driver", "configure ClickHouse connection", "add retry to ClickHouse", or works with ClickHouse JDBC, HTTP client, connection pooling, retry policies, or query execution wrappers. Covers 12 rules across connection management, retry handling, query execution, and observability. Do NOT use for ClickHouse SQL query optimization or schema design — use clickhouse-best-practices for that.
license: Apache-2.0
compatibility: Requires ClickHouse 24.1+. Patterns are Spring Boot / Java focused but principles apply to any language.
metadata:
  author: ClearTax
  version: "0.1.0"
---

# ClickHouse Driver Best Practices

12 rules for building reliable ClickHouse client libraries and integrations — covering connection management, retry handling, query execution, and observability.

## IMPORTANT: How to Apply This Skill

**Before building or reviewing ClickHouse client code, follow this priority order:**

1. **Check for applicable rules** in the `rules/` directory
2. **If rules exist:** Apply them and cite them using "Per `rule-name`..."
3. **If no rule exists:** Use general distributed systems / database driver knowledge
4. **Always cite your source:** rule name or general guidance

**Why this matters:** ClickHouse uses HTTP/native protocols with unique behaviors (async server-side execution, query_log tracking, LZ4 compression, MergeTree-specific error codes) where standard JDBC/database driver patterns are insufficient.

## Rule Categories by Priority

| Priority | Category | Impact | Prefix | Rules |
|----------|----------|--------|--------|-------|
| 1 | Retry & Resilience | CRITICAL | `retry-` | 3 |
| 2 | Connection Management | CRITICAL | `conn-` | 3 |
| 3 | Query Execution | HIGH | `query-` | 3 |
| 4 | Observability | HIGH | `obs-` | 3 |

## When to Apply

This skill activates when you encounter:

- ClickHouse client library or driver code
- Spring Boot ClickHouse integration setup
- JDBC or HTTP connection configuration for ClickHouse
- Retry policies for database operations
- Query timeout handling
- ClickHouse connection pooling or multi-tenant setup
- Query logging or performance monitoring for ClickHouse

**Do NOT activate for:** ClickHouse SQL optimization, schema design, or query tuning — use `clickhouse-best-practices` for that.

## Examples

### Example 1: Add retry logic to ClickHouse JDBC

User says: "Add retry handling to our ClickHouse queries"

1. Read `rules/retry-only-transient-errors.md` — only retry network/connection errors
2. Read `rules/retry-use-transparent-proxy.md` — apply retry via proxy, not caller changes
3. Read `rules/retry-structured-logging.md` — log WARN per attempt, ERROR on exhaustion

Result: Retry policy that retries `CannotGetJdbcConnectionException` and network errors, with fixed backoff and structured logging.

### Example 2: Handle mutation timeouts

User says: "Our ClickHouse inserts sometimes timeout but the data still gets written"

1. Read `rules/query-assign-mutation-id.md` — always assign UUID queryId to mutations
2. Read `rules/query-poll-on-timeout.md` — poll system.query_log on timeout, don't fail
3. Read `rules/obs-log-query-timing.md` — log query execution time for diagnosis

Result: Mutation wrapper that assigns queryId, catches timeout (error 159), and polls query_log for up to 5 minutes.

## Common Mistakes to Watch For

- **Retrying all errors**: Retrying query syntax errors or constraint violations wastes resources and delays failure
- **No queryId on mutations**: Without a queryId, you cannot determine if a timed-out write succeeded server-side
- **Failing on client timeout**: A client-side read timeout (error 159) does NOT mean the server operation failed
- **Same timeout for all paths**: JDBC and HTTP paths need independent timeout configuration
- **Not closing responses**: ClickHouseResponse objects must always be closed to prevent resource leaks
- **Index-based column access**: Using column indices instead of names makes code brittle to schema changes

## For Formal Reviews

When reviewing ClickHouse driver code, consult `references/review-checklist.md` for the full checklist.

## Performance Notes

- Take your time to verify retry policies only cover transient errors
- Check that every mutation path assigns a queryId
- Verify all ClickHouseResponse objects are closed in finally/usingWhen blocks

## Rule File Structure

Each rule file in `rules/` contains:

- **YAML frontmatter**: title, impact level, tags
- **Brief explanation**: Why this rule matters
- **Incorrect example**: Anti-pattern with explanation
- **Correct example**: Best practice with explanation
- **Additional context**: Trade-offs, when to apply
