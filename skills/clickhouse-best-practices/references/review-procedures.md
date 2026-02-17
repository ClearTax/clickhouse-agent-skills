# Review Procedures

Use these checklists when performing formal reviews of ClickHouse schemas, queries, or data ingestion pipelines.

## Schema Reviews (CREATE TABLE, ALTER TABLE)

**Read these rule files in order:**

1. `rules/schema-pk-plan-before-creation.md` - ORDER BY is immutable
2. `rules/schema-pk-cardinality-order.md` - Column ordering in keys
3. `rules/schema-pk-prioritize-filters.md` - Filter column inclusion
4. `rules/schema-types-native-types.md` - Proper type selection
5. `rules/schema-types-minimize-bitwidth.md` - Numeric type sizing
6. `rules/schema-types-lowcardinality.md` - LowCardinality usage
7. `rules/schema-types-avoid-nullable.md` - Nullable vs DEFAULT
8. `rules/schema-partition-low-cardinality.md` - Partition count limits
9. `rules/schema-partition-lifecycle.md` - Partitioning purpose

**Check for:**
- [ ] PRIMARY KEY / ORDER BY column order (low-to-high cardinality)
- [ ] Data types match actual data ranges
- [ ] LowCardinality applied to appropriate string columns
- [ ] Partition key cardinality bounded (100-1,000 values)
- [ ] ReplacingMergeTree has version column if used

## Query Reviews (SELECT, JOIN, aggregations)

**Read these rule files:**

1. `rules/query-join-choose-algorithm.md` - Algorithm selection
2. `rules/query-join-filter-before.md` - Pre-join filtering
3. `rules/query-join-use-any.md` - ANY vs regular JOIN
4. `rules/query-index-skipping-indices.md` - Secondary index usage
5. `rules/schema-pk-filter-on-orderby.md` - Filter alignment with ORDER BY

**Check for:**
- [ ] Filters use ORDER BY prefix columns
- [ ] JOINs filter tables before joining (not after)
- [ ] Correct JOIN algorithm for table sizes
- [ ] Skipping indices for non-ORDER BY filter columns

## Insert Strategy Reviews (data ingestion, updates, deletes)

**Read these rule files:**

1. `rules/insert-batch-size.md` - Batch sizing requirements
2. `rules/insert-mutation-avoid-update.md` - UPDATE alternatives
3. `rules/insert-mutation-avoid-delete.md` - DELETE alternatives
4. `rules/insert-async-small-batches.md` - Async insert usage
5. `rules/insert-optimize-avoid-final.md` - OPTIMIZE TABLE risks

**Check for:**
- [ ] Batch size 10K-100K rows per INSERT
- [ ] No ALTER TABLE UPDATE for frequent changes
- [ ] ReplacingMergeTree or CollapsingMergeTree for update patterns
- [ ] Async inserts enabled for high-frequency small batches
