---
title: Always Assign a UUID queryId to Mutations
impact: CRITICAL
impactDescription: "Enables post-hoc status checking when mutations timeout"
tags: [driver, query, mutation, idempotency]
---

## Always Assign a UUID queryId to Mutations

**Impact: CRITICAL** (without queryId, timed-out mutations are unrecoverable)

Every ClickHouse mutation (INSERT, ALTER, DELETE) must be assigned a unique `queryId`. This enables you to check whether a timed-out mutation actually completed on the server by querying `system.query_log`.

**Incorrect (no queryId — blind to mutation outcome on timeout):**

```java
public long executeMutation(String query) {
    try (ClickHouseResponse response = client.read(node)
            .query(query)
            .executeAndWait()) {
        return response.getSummary().getWrittenRows();
    } catch (Exception e) {
        // Timeout? Did the mutation succeed? We'll never know.
        throw new RuntimeException(e);
    }
}
```

**Correct (UUID queryId enables status tracking):**

```java
public Mono<Long> executeMutation(String query) {
    String queryId = UUID.randomUUID().toString();

    return Mono.fromFuture(
        client.write(node)
            .query(query, queryId)
            .format(ClickHouseFormat.RowBinary)
            .execute()
            .thenApply(response -> {
                long rows = response.getSummary().getWrittenRows();
                response.close();
                return rows;
            })
    ).onErrorResume(e -> {
        if (isTimeoutError(e)) {
            // Mutation may still be running server-side — poll for status
            return pollQueryStatus(queryId);
        }
        return Mono.error(e);
    });
}

private boolean isTimeoutError(Throwable e) {
    if (e instanceof ExecutionException ee) e = ee.getCause();
    return e instanceof ClickHouseException che
        && che.getErrorCode() == 159; // Read timeout
}
```

**Why this matters:**
- ClickHouse executes mutations server-side even after the client disconnects
- A client-side timeout (error 159) does NOT cancel the server-side operation
- Without a queryId, you cannot look up the mutation in `system.query_log`
- Duplicate mutations without idempotency guarantees can corrupt data

**Always use UUID v4** — it provides sufficient uniqueness for concurrent mutations.
