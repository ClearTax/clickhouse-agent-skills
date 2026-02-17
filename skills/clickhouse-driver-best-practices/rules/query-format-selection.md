---
title: Use Appropriate Binary Formats for Reads vs Writes
impact: HIGH
impactDescription: "Wrong format wastes bandwidth or loses column name access"
tags: [driver, query, format, performance]
---

## Use Appropriate Binary Formats for Reads vs Writes

**Impact: HIGH** (affects both performance and code maintainability)

ClickHouse supports multiple data formats. Use `RowBinaryWithNamesAndTypes` for reads (provides column names) and `RowBinary` for writes (minimal overhead).

**Incorrect (same format for everything):**

```java
// Using JSON for everything — slow and bloated
ClickHouseResponse response = client.read(node)
    .query("SELECT user_id, event_type FROM events")
    .format(ClickHouseFormat.JSONEachRow)  // 3-5x larger than binary
    .executeAndWait();
```

**Correct (format matches operation type):**

```java
// Reads: RowBinaryWithNamesAndTypes — enables named column access
public Mono<ClickHouseResponse> executeReadQuery(String query) {
    return Mono.fromFuture(
        client.read(node)
            .query(query)
            .format(ClickHouseFormat.RowBinaryWithNamesAndTypes)
            .execute()
    );
}

// Writes: RowBinary — compact, no metadata overhead
public Mono<Long> executeMutation(String query, String queryId) {
    return Mono.fromFuture(
        client.write(node)
            .query(query, queryId)
            .format(ClickHouseFormat.RowBinary)
            .execute()
            .thenApply(r -> r.getSummary().getWrittenRows())
    );
}
```

**Format selection guide:**

| Operation | Format | Why |
|-----------|--------|-----|
| Read queries | `RowBinaryWithNamesAndTypes` | Named column access: `record.getValue("type")` |
| Mutations / Inserts | `RowBinary` | Minimal overhead, no metadata needed |
| Debugging / Ad-hoc | `JSONEachRow` | Human readable, but 3-5x larger |
| Bulk export | `Native` | Most compact, fastest for large datasets |

**Why named column access matters:**
- `record.getValue("type")` survives schema changes (column reorder)
- `record.getValue(0)` breaks when columns are added or reordered
- Named access is self-documenting — no need to track column positions

**Include LZ4 dependency** for transparent HTTP compression:

```xml
<dependency>
    <groupId>net.jpountz.lz4</groupId>
    <artifactId>lz4</artifactId>
    <version>1.3.0</version>
</dependency>
```
