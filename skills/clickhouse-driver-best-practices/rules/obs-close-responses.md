---
title: Always Close ClickHouseResponse Objects
impact: HIGH
impactDescription: "Unclosed responses leak connections and memory"
tags: [driver, observability, resource-management]
---

## Always Close ClickHouseResponse Objects

**Impact: HIGH** (resource leaks cause connection pool exhaustion under load)

`ClickHouseResponse` objects hold network connections and buffers. They must always be closed, even on error paths. Use `Mono.usingWhen()` in reactive code or try-with-resources in synchronous code.

**Incorrect (response not closed on error):**

```java
public <T> Mono<T> executeAndProcess(String query, Function<ClickHouseResponse, T> mapper) {
    return Mono.fromFuture(client.read(node).query(query).execute())
        .map(response -> {
            T result = mapper.apply(response);
            response.close();  // Never reached if mapper throws
            return result;
        });
}
```

**Correct (usingWhen guarantees cleanup):**

```java
public <T> Mono<T> executeAndProcess(
        String query, Function<ClickHouseResponse, Mono<T>> action) {
    return Mono.usingWhen(
        // Resource acquisition
        Mono.fromFuture(client.read(node).query(query).execute()),
        // Use the resource
        action,
        // Cleanup — always called
        response -> Mono.fromRunnable(response::close)
    );
}
```

**For synchronous code, use try-with-resources:**

```java
public long executeCount(String query) {
    try (ClickHouseResponse response = client.read(node)
            .query(query)
            .format(ClickHouseFormat.RowBinaryWithNamesAndTypes)
            .executeAndWait()) {
        return response.firstRecord().getValue(0).asLong();
    }
}
```

**Patterns by context:**

| Context | Pattern |
|---------|---------|
| Reactive (Mono/Flux) | `Mono.usingWhen()` with cleanup function |
| Synchronous | try-with-resources |
| CompletableFuture | `.whenComplete((r, e) -> { if (r != null) r.close(); })` |

**Never rely on garbage collection** to close responses — GC timing is unpredictable and connection pools have finite capacity.
