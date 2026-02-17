---
title: Configure Separate Timeouts for Each Connection Path
impact: HIGH
impactDescription: "Different protocols have different latency characteristics"
tags: [driver, connection, timeout, configuration]
---

## Configure Separate Timeouts for Each Connection Path

**Impact: HIGH** (shared timeouts cause either premature failures or hung connections)

JDBC and HTTP client paths to ClickHouse have different latency profiles. The HTTP path (used for async/reactive) often handles large mutations that need longer timeouts. The JDBC path (used for queries) typically needs shorter timeouts.

**Incorrect (single timeout for everything):**

```yaml
# One timeout for all paths — too short for mutations, too long for queries
clickhouse:
  timeout: 30000
```

**Correct (per-path timeouts):**

```yaml
clickhouse:
  host: clickhouse.example.com
  port: 8123
  database: analytics
  socket-timeout: 30000          # JDBC path — queries
  direct-socket-timeout: 60000   # HTTP path — mutations, large writes
  enable-ssl: true
```

```java
// JDBC DataSource with its own timeout
@Bean
public DataSource clickHouseDataSource(
        @Value("${clickhouse.socket-timeout:30000}") int socketTimeout) {
    Properties props = new Properties();
    props.setProperty("socket_timeout", String.valueOf(socketTimeout));
    return new ClickHouseDataSource(jdbcUrl, props);
}

// HTTP client with its own timeout
@Bean
public ClickHouseNode clickHouseNode(
        @Value("${clickhouse.direct-socket-timeout:60000}") int directSocketTimeout) {
    Map<String, String> options = new HashMap<>();
    options.put(ClickHouseClientOption.SOCKET_TIMEOUT.getKey(),
        String.valueOf(directSocketTimeout));
    return ClickHouseNode.builder().host(host).options(options).build();
}
```

**Guidelines:**
- JDBC queries: 30s default (most queries should complete faster)
- HTTP mutations: 60s+ (large batch inserts take longer)
- Polling for timed-out mutations: separate 5-minute cap (see `query-poll-on-timeout`)

**Important:** Keep SSL configuration consistent across both paths. An inconsistency (e.g., SSL on for JDBC, off for HTTP) causes hard-to-debug connection failures.
