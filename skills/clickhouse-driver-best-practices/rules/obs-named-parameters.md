---
title: Use Named Parameters for Query Safety and Clarity
impact: HIGH
impactDescription: "Prevents SQL injection and makes queries self-documenting"
tags: [driver, observability, query, security]
---

## Use Named Parameters for Query Safety and Clarity

**Impact: HIGH** (string concatenation risks SQL injection and is hard to debug)

Always use named parameters or parameterized queries instead of string concatenation. This applies to both JDBC (`NamedParameterJdbcTemplate`) and native client (`ClickHouseParameterizedQuery`) paths.

**Incorrect (string concatenation — injection risk):**

```java
// SQL injection vulnerability + hard to read
String query = "SELECT * FROM events WHERE user_id = " + userId
    + " AND event_date >= '" + startDate + "'";
jdbcTemplate.query(query, mapper);
```

**Correct (named parameters — JDBC path):**

```java
String query = "SELECT * FROM events WHERE user_id = :userId AND event_date >= :startDate";
Map<String, Object> params = Map.of(
    "userId", userId,
    "startDate", startDate
);
namedParameterJdbcTemplate.query(query, params, mapper);
```

**Correct (named parameters — native HTTP client path):**

```java
String query = "SELECT count() FROM events WHERE user_id = :userId";
Map<String, String> params = Map.of("userId", String.valueOf(userId));

return Mono.fromFuture(
    client.read(node)
        .query(ClickHouseParameterizedQuery.apply(query, params))
        .format(ClickHouseFormat.RowBinaryWithNamesAndTypes)
        .execute()
);
```

**Benefits:**
- Prevents SQL injection
- Query text is constant — easier to find in query_log
- Parameters are typed — no quoting errors
- Queries are self-documenting with meaningful parameter names

**For the JDBC path:** Use `NamedParameterJdbcOperations` instead of `JdbcOperations` when queries have parameters. Register both as beans.
