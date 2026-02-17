---
title: Use Annotation-Driven Bean Activation
impact: CRITICAL
impactDescription: "Consumers opt-in to exactly the features they need"
tags: [driver, connection, spring, configuration]
---

## Use Annotation-Driven Bean Activation

**Impact: CRITICAL** (prevents unwanted beans and startup failures)

ClickHouse connection beans should be activated via explicit annotations (`@EnableClickhouseJdbcTemplate`, etc.), not via Spring Boot auto-configuration. This lets consumers choose their exact connection mode.

**Incorrect (auto-configuration registers everything):**

```java
// Auto-configures ALL connection types — consumers get beans they don't need
@Configuration
@AutoConfiguration
@ConditionalOnClass(ClickHouseDataSource.class)
public class ClickHouseAutoConfig {
    @Bean public ClickHouseClient client() { ... }
    @Bean public JdbcTemplate jdbcTemplate() { ... }
    @Bean public JdbcOperations retryableJdbc() { ... }
}
```

**Correct (annotation imports only what's needed):**

```java
// Annotation activates specific configuration
@Import({ ClickHouseJdbcDataSourceConfig.class, ClickHouseRetryableJdbcTemplate.class })
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface EnableRetryableClickhouseJdbcTemplate { }
```

```java
// Consumer opts-in to exactly the mode they need
@Configuration
@EnableRetryableClickhouseJdbcTemplate
public class MyAppConfig { }
```

**Connection modes to support:**
1. **Direct HTTP client** — for reactive/async patterns with native ClickHouse client
2. **JDBC Template** — for standard Spring JDBC with query logging
3. **Retryable JDBC Template** — for JDBC with transparent retry via proxy

**Guard with `@ConditionalOnProperty`:**

```java
@Bean
@ConditionalOnProperty(name = "clickhouse.enabled", havingValue = "true", matchIfMissing = true)
public ClickHouseClient clickHouseClient() { ... }
```

This ensures beans are enabled by default but can be disabled with `clickhouse.enabled=false`.
