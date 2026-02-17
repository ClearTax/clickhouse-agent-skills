# Driver Code Review Checklist

Use this checklist when reviewing ClickHouse client/driver code.

## Connection Management

- [ ] Connection beans activated via explicit annotations, not auto-configuration
- [ ] `@ConditionalOnProperty` guards bean creation with `clickhouse.enabled`
- [ ] Utility services use `Optional<>` injection for conditional ClickHouse beans
- [ ] Separate timeout configuration for JDBC and HTTP paths
- [ ] SSL configuration is consistent across both connection paths
- [ ] LZ4 dependency included for HTTP compression

## Retry & Resilience

- [ ] Retry policy only covers transient errors (connection, network)
- [ ] Query syntax errors, auth failures, and constraint violations are NOT retried
- [ ] Retry applied via transparent proxy, not in individual callers
- [ ] `InvocationTargetException` is unwrapped before error type checking
- [ ] Fixed backoff configured (default: 1000ms between attempts)
- [ ] Max retry attempts configured (default: 3)
- [ ] `throwLastExceptionOnExhausted` is `true` — callers see real exceptions
- [ ] Retry context caching disabled (`setRetryContextCache(null)`)

## Query Execution

- [ ] Every mutation has a UUID queryId assigned
- [ ] Timeout errors (code 159) trigger query_log polling, not immediate failure
- [ ] Polling has a hard time cap (e.g., 5 minutes)
- [ ] Reads use `RowBinaryWithNamesAndTypes` for named column access
- [ ] Writes use `RowBinary` for minimal overhead
- [ ] Named parameters used instead of string concatenation
- [ ] `allowMultiQueries=true` set in JDBC URL if needed

## Observability

- [ ] All queries logged with SQL text and execution timing
- [ ] Retry attempts logged at WARN level
- [ ] Retry exhaustion logged at ERROR level
- [ ] All `ClickHouseResponse` objects closed (usingWhen or try-with-resources)
- [ ] All `prepareStatement` overloads intercepted (5 variants)

## Multi-Tenant (if applicable)

- [ ] Each tenant has its own `JdbcOperations` bean
- [ ] Tenant beans accessible via `Map<String, JdbcOperations>`
- [ ] Tenant configuration under `clickhouse.multi-tenant.clickhouse-tenants[n].*`
