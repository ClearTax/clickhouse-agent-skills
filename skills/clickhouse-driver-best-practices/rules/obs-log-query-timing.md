---
title: Log All Queries with Execution Timing
impact: HIGH
impactDescription: "Enables performance diagnosis and slow query detection"
tags: [driver, observability, logging, performance]
---

## Log All Queries with Execution Timing

**Impact: HIGH** (without timing data, slow queries are invisible until they cause incidents)

Wrap query execution in a timing interceptor that logs the SQL text and elapsed time. Use a DataSource proxy to apply this transparently to all queries.

**Incorrect (no query logging — blind to performance):**

```java
@Bean
public JdbcTemplate jdbcTemplate(DataSource ds) {
    return new JdbcTemplate(ds); // No visibility into query performance
}
```

**Correct (DataSource proxy with timing):**

```java
public class QueryLoggingInterceptor extends DataSourceAdapter {
    private final DataSource delegate;

    public QueryLoggingInterceptor(DataSource delegate) {
        this.delegate = delegate;
    }

    @Override
    public Connection getConnection() throws SQLException {
        return new ConnectionProxy(delegate.getConnection());
    }

    private class ConnectionProxy extends ConnectionAdapter {
        @Override
        public PreparedStatement prepareStatement(String sql) throws SQLException {
            StopWatch stopWatch = new StopWatch();
            stopWatch.start();
            PreparedStatement ps = delegate.prepareStatement(sql);
            stopWatch.stop();
            log.info("SQL query [{}] prepared in [{}] ms",
                sql, stopWatch.getTotalTimeMillis());
            return ps;
        }
    }
}
```

```java
@Bean
public JdbcTemplate jdbcTemplate(DataSource dataSource) {
    DataSource proxied = new QueryLoggingInterceptor(dataSource);
    return new JdbcTemplate(proxied);
}
```

**What to log:**
- SQL query text (parameterized, not with values — avoid leaking data)
- Execution time in milliseconds
- Log at INFO level for normal queries
- Consider WARN for queries exceeding a threshold (e.g., > 5 seconds)

**Important:** Override ALL `prepareStatement` overloads — there are five variants with different parameter signatures. Missing any one creates a blind spot.
