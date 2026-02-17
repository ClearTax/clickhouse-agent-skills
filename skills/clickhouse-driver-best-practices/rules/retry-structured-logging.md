---
title: Log Retries with Structured Severity Levels
impact: HIGH
impactDescription: "Distinguishes transient blips from persistent failures in alerting"
tags: [driver, retry, logging, observability]
---

## Log Retries with Structured Severity Levels

**Impact: HIGH** (prevents alert fatigue while ensuring real failures are visible)

Retry logging must use different severity levels for individual attempts vs. final exhaustion. Logging every retry at ERROR triggers false alerts; logging exhaustion at WARN misses real outages.

**Incorrect (same level for all retry events):**

```java
// All retries at ERROR — floods alerts during brief network blips
retryTemplate.registerListener(new RetryListenerSupport() {
    @Override
    public <T, E extends Throwable> void onError(
            RetryContext context, RetryCallback<T, E> callback, Throwable t) {
        log.error("ClickHouse query failed, retrying...", t);
    }
});
```

**Correct (graduated severity):**

```java
public class ClickhouseRetryListener extends RetryListenerSupport {

    @Override
    public <T, E extends Throwable> boolean open(
            RetryContext context, RetryCallback<T, E> callback) {
        if (context.getLastThrowable() != null) {
            log.warn("ClickHouse retry context opened with exception: {}",
                context.getLastThrowable().getMessage());
        }
        return true; // Allow retry to proceed
    }

    @Override
    public <T, E extends Throwable> void onError(
            RetryContext context, RetryCallback<T, E> callback, Throwable t) {
        log.warn("ClickHouse operation failed, attempt #{}: {}",
            context.getRetryCount(), t.getMessage());
    }

    @Override
    public <T, E extends Throwable> void close(
            RetryContext context, RetryCallback<T, E> callback) {
        if (context.isExhaustedOnly()) {
            log.error("ClickHouse retries exhausted after {} attempts: {}",
                context.getRetryCount(),
                context.getLastThrowable().getMessage());
        }
    }
}
```

**Severity model:**
- **WARN** — individual retry attempt failed (transient, expected during brief outages)
- **ERROR** — all retries exhausted (persistent failure, needs investigation)

**Additional:** Disable retry context caching with `retryTemplate.setRetryContextCache(null)` to ensure fresh exception evaluation on each attempt.
