---
title: Poll system.query_log on Timeout Instead of Failing
impact: CRITICAL
impactDescription: "Prevents false negatives — a timed-out mutation may have succeeded"
tags: [driver, query, timeout, polling]
---

## Poll system.query_log on Timeout Instead of Failing

**Impact: CRITICAL** (failing on timeout can trigger redundant retries of successful mutations)

When a ClickHouse mutation times out (error code 159), the server may still be executing it. Instead of failing immediately, poll `system.query_log` using the mutation's queryId to determine the actual outcome.

**Incorrect (fail immediately on timeout):**

```java
try {
    client.write(node).query(mutation, queryId).execute().get();
} catch (ExecutionException e) {
    // Timeout = failure? Not necessarily — the mutation may still be running
    throw new RuntimeException("Mutation failed", e);
}
```

**Correct (poll query_log to determine actual status):**

```java
private Mono<Long> pollQueryStatus(String queryId) {
    String statusQuery = String.format(
        "SELECT type FROM system.query_log WHERE query_id = '%s'", queryId);

    return Flux.interval(Duration.ofSeconds(1))
        .take(Duration.ofMinutes(5))           // Hard cap: 5 minutes
        .flatMap(tick -> checkQueryStatus(statusQuery))
        .takeUntil(status -> status != QueryStatus.INPROGRESS)
        .last()
        .map(status -> status == QueryStatus.FINISH ? 1L : -1L);
}

private Mono<QueryStatus> checkQueryStatus(String statusQuery) {
    return executeReadQuery(statusQuery)
        .map(response -> {
            ClickHouseRecord record = response.firstRecord();
            if (record.size() == 0) return QueryStatus.INPROGRESS;

            String type = record.getValue("type").asString();
            if ("QueryFinish".equals(type)) return QueryStatus.FINISH;
            if (type.startsWith("Exception"))  return QueryStatus.ERROR;
            return QueryStatus.INPROGRESS;
        });
}

public enum QueryStatus { FINISH, ERROR, INPROGRESS }
```

**Polling strategy:**
- **Interval:** 1 second between checks
- **Hard cap:** 5 minutes maximum (prevents infinite polling)
- **Terminal states:** `QueryFinish` = success, `Exception*` = failure
- **Non-terminal:** Empty result or other types = still in progress

**When to poll:**
- Only on error code 159 (client read timeout)
- Only for mutations (INSERT, ALTER, DELETE)
- NOT for read queries — reads that timeout should be retried from scratch

**Important:** The `system.query_log` is flushed asynchronously. There may be a brief delay before a completed query appears. The 1-second polling interval accounts for this.
