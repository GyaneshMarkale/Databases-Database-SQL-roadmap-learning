# Module 5 — Execution Plans and Query Optimization

## Outcomes

- Read EXPLAIN output and distinguish estimates from actual observations.
- Recognize scan and join nodes.
- Diagnose row-estimation errors, repeated work, sorts and buffer activity.
- Tune using a measurable hypothesis rather than guessing.

## 1. EXPLAIN versus EXPLAIN ANALYZE

    EXPLAIN
    SELECT order_id, created_at
    FROM orders
    WHERE customer_id = 42001;

EXPLAIN reports the planner's proposed operations and estimated costs.

    EXPLAIN (ANALYZE, BUFFERS)
    SELECT order_id, created_at
    FROM orders
    WHERE customer_id = 42001;

EXPLAIN ANALYZE executes the statement and reports actual observations, including row counts, loops, timings and buffer activity. Use it carefully: it executes INSERT/UPDATE/DELETE statements too. Test modifying statements in a safe environment or use a deliberate rollback strategy, while remembering rollback does not undo every possible external effect or sequence advancement.

## 2. Explain plan fields

- cost=startup..total: planner units used to compare alternatives, not milliseconds.
- rows: estimated output row count.
- width: estimated average row width in bytes.
- actual time: observed timing in milliseconds per loop.
- actual rows: observed output rows per loop, averaged when loops exceed one.
- loops: number of executions of the node.
- buffers: shared/local/temp buffer activity.
- planning and execution time: reported statement timings.

Read plans from the leaves upward to understand how child operations produce the parent's input. Total node times can be cumulative/overlapping; do not sum every node time as though each were independent.

## 3. Scan nodes

- Seq Scan: visits table pages and tests rows. It can be optimal for small tables or broad predicates.
- Index Scan: navigates an index and fetches table rows.
- Index Only Scan: may serve required columns from the index; PostgreSQL may still perform heap fetches for visibility.
- Bitmap Index Scan + Bitmap Heap Scan: collects tuple locations and visits relevant heap pages; bitmap scans can combine multiple indexes.

A sequential scan is not automatically bad. Compare the work required to return the result.

## 4. Join nodes

- Nested Loop: for each outer row, probes the inner side. Often good when the outer input is small and the inner lookup is cheap; expensive when repeated many times.
- Hash Join: builds a hash table from one input and probes with another, commonly useful for substantial equality joins. Large hash operations can spill.
- Merge Join: processes inputs ordered by the join keys. Existing order can help; otherwise sorts may be required.

Choose by actual input sizes, filtering, join condition, available ordering/indexes and the planner's estimates—not by declaring one join type universally fastest.

## 5. Compare estimated and actual rows

If the plan expects 50 rows but returns 250,000, the cardinality estimate is far off. Investigate:
- stale statistics,
- skewed distributions,
- correlated predicates,
- expressions and difficult-to-estimate filters,
- parameter-sensitive plan choices.

Run ANALYZE table_name to refresh planner statistics; this command is different from EXPLAIN ANALYZE. PostgreSQL extended statistics can help with some dependencies or most-common-value combinations, but should be introduced after a demonstrated estimation issue.

## 6. Sorts, loops and buffers

A sort reporting external merge and temp writes may be spilling to disk. Consider whether the query sorts too many rows, whether a matching index can provide order, and whether filtering can reduce input before the sort.

A nested loop whose inner scan has loops=100000 may be expensive even if a single inner lookup is cheap. Check whether the outer input is unnecessarily large or estimates are wrong.

With BUFFERS:
- shared hit: page found in PostgreSQL shared buffers.
- shared read: page had to be read into shared buffers, though OS cache may satisfy it.
- temp read/written: temporary data read or written, often related to spill.
Buffer counts are cumulative in plan reporting and a buffer read is not synonymous with a physical disk read.

## 7. Query rewrite example

Less index-friendly with a normal index on created_at:

    SELECT *
    FROM orders
    WHERE DATE(created_at) = DATE '2026-11-07';

A range predicate often aligns better with a B-tree:

    SELECT order_id, created_at, total_amount
    FROM orders
    WHERE created_at >= TIMESTAMPTZ '2026-11-07 00:00:00+00'
      AND created_at <  TIMESTAMPTZ '2026-11-08 00:00:00+00';

The example uses UTC day boundaries. Calculate boundaries in the intended business timezone. An expression index is another option if the exact expression is a stable, common access pattern.

## 8. Repeatable tuning process

1. Capture the exact query, parameters, latency and invocation frequency.
2. Reproduce representative data volume and distribution.
3. Inspect EXPLAIN; use EXPLAIN ANALYZE, BUFFERS safely.
4. Identify where actual rows, loops, sorts, buffer activity or temp writes accumulate.
5. Write one testable hypothesis.
6. Change one thing: index, statistics, query, schema or data access.
7. Compare plans, latency and buffers before/after.
8. Test write overhead and other query regressions.
9. Document results and revisit when workload changes.

## References

- [PostgreSQL — EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
- [PostgreSQL — EXPLAIN command details](https://www.postgresql.org/docs/current/sql-explain.html)
- [PostgreSQL — Planner statistics](https://www.postgresql.org/docs/current/planner-stats.html)
- [PostgreSQL — Routine vacuuming and statistics maintenance](https://www.postgresql.org/docs/current/routine-vacuuming.html)
