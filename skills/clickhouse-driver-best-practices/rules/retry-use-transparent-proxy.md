---
title: Apply Retry Transparently via Proxy
impact: CRITICAL
impactDescription: "Eliminates retry boilerplate across all callers"
tags: [driver, retry, proxy, spring]
---

## Apply Retry Transparently via Proxy

**Impact: CRITICAL** (prevents inconsistent retry behavior across codebase)

Retry logic should be applied at the infrastructure layer using a dynamic proxy, not sprinkled across every caller. This ensures every database operation gets consistent retry behavior without code changes.

**Incorrect (retry in every caller):**

```java
// Every service method must remember to add retry
@Service
public class EventService {
    public List<Event> getEvents() {
        return retryTemplate.execute(ctx ->
            jdbcTemplate.query("SELECT * FROM events", eventMapper));
    }

    public void insertEvent(Event e) {
        retryTemplate.execute(ctx ->
            jdbcTemplate.update("INSERT INTO events ...", e.getId()));
    }

    // Easy to forget retry on new methods...
}
```

**Correct (transparent proxy wraps all operations):**

```java
@Bean
public JdbcOperations retryableJdbcOperations(
        JdbcTemplate jdbcTemplate, RetryTemplate retryTemplate) {
    return (JdbcOperations) Proxy.newProxyInstance(
        JdbcOperations.class.getClassLoader(),
        new Class[] { JdbcOperations.class },
        (proxy, method, args) ->
            retryTemplate.execute(context -> method.invoke(jdbcTemplate, args))
    );
}
```

```java
// Callers use JdbcOperations as normal — retry is invisible
@Service
public class EventService {
    private final JdbcOperations jdbc; // Injected retryable proxy

    public List<Event> getEvents() {
        return jdbc.query("SELECT * FROM events", eventMapper);
    }
}
```

**Key benefits:**
- Every operation gets retry automatically — no missed methods
- Callers don't know about retry — clean separation of concerns
- Single place to change retry configuration
- Works with `JdbcOperations` and `NamedParameterJdbcOperations` interfaces

**Important:** Set `retryTemplate.setThrowLastExceptionOnExhausted(true)` so callers see the real exception, not a retry framework wrapper.
