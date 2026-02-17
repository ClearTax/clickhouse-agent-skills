---
title: Use Optional Bean Injection for Conditional Dependencies
impact: HIGH
impactDescription: "Prevents NoSuchBeanDefinitionException when ClickHouse is disabled"
tags: [driver, connection, spring, dependency-injection]
---

## Use Optional Bean Injection for Conditional Dependencies

**Impact: HIGH** (prevents application startup failures)

Utility services that depend on ClickHouse beans must use `Optional<>` injection so they survive when ClickHouse is disabled. Without this, disabling ClickHouse (`clickhouse.enabled=false`) crashes the application.

**Incorrect (hard dependency — fails if beans are missing):**

```java
@Service
public class ClickhouseHttpUtil {
    private final ClickHouseClient client;
    private final ClickHouseNode node;

    // Fails with NoSuchBeanDefinitionException if ClickHouse is disabled
    public ClickhouseHttpUtil(ClickHouseClient client, ClickHouseNode node) {
        this.client = client;
        this.node = node;
    }
}
```

**Correct (optional injection — gracefully handles missing beans):**

```java
@Service
public class ClickhouseHttpUtil {
    private final ClickHouseClient client;
    private final ClickHouseNode node;

    public ClickhouseHttpUtil(
            @Qualifier("clickHouseHttpClient") Optional<ClickHouseClient> clickHouseClient,
            @Qualifier("clickHouseNode") Optional<ClickHouseNode> clickHouseNode) {
        this.client = clickHouseClient.orElse(null);
        this.node = clickHouseNode.orElse(null);
    }

    public Mono<Long> executeQuery(String query) {
        if (client == null || node == null) {
            return Mono.error(new IllegalStateException("ClickHouse is not configured"));
        }
        // ... execute query
    }
}
```

**When to use Optional injection:**
- Utility/helper services that might be used across multiple modules
- Services in shared libraries where ClickHouse may or may not be enabled
- Any bean that depends on conditionally-created ClickHouse beans

**When NOT to use it:**
- Application-specific services that always require ClickHouse — let the hard failure surface the misconfiguration early
