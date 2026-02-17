---
title: Only Retry Transient Network and Connection Errors
impact: CRITICAL
impactDescription: "Retrying non-transient errors wastes resources and delays failure detection"
tags: [driver, retry, error-handling]
---

## Only Retry Transient Network and Connection Errors

**Impact: CRITICAL** (wrong retry scope causes cascading failures)

ClickHouse retry policies must distinguish transient errors (network failures, connection drops) from permanent errors (syntax errors, constraint violations). Retrying permanent errors wastes time and can mask real bugs.

**Incorrect (retry all exceptions):**

```java
// Retries EVERYTHING — syntax errors, auth failures, constraint violations
RetryTemplate retryTemplate = new RetryTemplate();
retryTemplate.setRetryPolicy(new SimpleRetryPolicy(3));
retryTemplate.execute(context -> jdbcTemplate.query(sql, mapper));
```

**Correct (retry only transient errors):**

```java
public class ClickHouseRetryPolicy extends SimpleRetryPolicy {
    @Override
    public boolean canRetry(RetryContext context) {
        if (!super.canRetry(context)) return false;

        Throwable t = context.getLastThrowable();
        if (t == null) return true;

        // Unwrap proxy invocation exceptions
        if (t instanceof InvocationTargetException ite) {
            t = ite.getCause();
        }

        // Only retry connection and network errors
        return t instanceof CannotGetJdbcConnectionException
            || (t instanceof ClickHouseException che
                && che.getErrorCode() == ClickHouseException.ERROR_NETWORK)
            || t instanceof DataAccessResourceFailureException;
    }
}
```

**Retryable errors (whitelist):**
- `CannotGetJdbcConnectionException` — connection pool exhaustion or network failure
- `ClickHouseException` with `ERROR_NETWORK` — transport-layer failure
- `DataAccessResourceFailureException` — resource unavailable

**Non-retryable errors (let them propagate):**
- Query syntax errors
- Authentication failures
- Constraint violations
- ClickHouse server-side errors (non-network)

Reference: [Spring Retry Documentation](https://docs.spring.io/spring-retry/docs/api/current/)
