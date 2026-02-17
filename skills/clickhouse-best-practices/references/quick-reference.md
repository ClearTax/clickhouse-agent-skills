# Quick Reference

All 28 rules organized by category and priority.

## Schema Design - Primary Key (CRITICAL)

- `schema-pk-plan-before-creation` - Plan ORDER BY before table creation (immutable)
- `schema-pk-cardinality-order` - Order columns low-to-high cardinality
- `schema-pk-prioritize-filters` - Include frequently filtered columns
- `schema-pk-filter-on-orderby` - Query filters must use ORDER BY prefix

## Schema Design - Data Types (CRITICAL)

- `schema-types-native-types` - Use native types, not String for everything
- `schema-types-minimize-bitwidth` - Use smallest numeric type that fits
- `schema-types-lowcardinality` - LowCardinality for <10K unique strings
- `schema-types-enum` - Enum for finite value sets with validation
- `schema-types-avoid-nullable` - Avoid Nullable; use DEFAULT instead

## Schema Design - Partitioning (HIGH)

- `schema-partition-low-cardinality` - Keep partition count 100-1,000
- `schema-partition-lifecycle` - Use partitioning for data lifecycle, not queries
- `schema-partition-query-tradeoffs` - Understand partition pruning trade-offs
- `schema-partition-start-without` - Consider starting without partitioning

## Schema Design - JSON (MEDIUM)

- `schema-json-when-to-use` - JSON for dynamic schemas; typed columns for known

## Query Optimization - JOINs (CRITICAL)

- `query-join-choose-algorithm` - Select algorithm based on table sizes
- `query-join-use-any` - ANY JOIN when only one match needed
- `query-join-filter-before` - Filter tables before joining
- `query-join-consider-alternatives` - Dictionaries/denormalization vs JOIN
- `query-join-null-handling` - join_use_nulls=0 for default values

## Query Optimization - Indices (HIGH)

- `query-index-skipping-indices` - Skipping indices for non-ORDER BY filters

## Query Optimization - Materialized Views (HIGH)

- `query-mv-incremental` - Incremental MVs for real-time aggregations
- `query-mv-refreshable` - Refreshable MVs for complex joins

## Insert Strategy - Batching (CRITICAL)

- `insert-batch-size` - Batch 10K-100K rows per INSERT

## Insert Strategy - Async (HIGH)

- `insert-async-small-batches` - Async inserts for high-frequency small batches
- `insert-format-native` - Native format for best performance

## Insert Strategy - Mutations (CRITICAL)

- `insert-mutation-avoid-update` - ReplacingMergeTree instead of ALTER UPDATE
- `insert-mutation-avoid-delete` - Lightweight DELETE or DROP PARTITION

## Insert Strategy - Optimization (HIGH)

- `insert-optimize-avoid-final` - Let background merges work
