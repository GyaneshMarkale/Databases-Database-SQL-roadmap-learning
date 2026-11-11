# Module 3 — Indexes and Query Performance

## Outcomes

- Explain B-tree indexes, index scans, sequential scans and index-only scans.
- Design composite indexes based on predicates and sort order.
- Understand selectivity and why an optimizer may reject an index.
- Evaluate partial, expression and covering indexes.
- Account for write, storage and maintenance costs.

## 1. What an index does

An index is an auxiliary structure that helps the database locate or order rows. PostgreSQL B-tree indexes support equality, ranges and ordered retrieval. A lookup is often described as O(log N), but total query cost also includes row fetches, filtering, joins, sorting and I/O.

    CREATE INDEX idx_orders_customer
        ON orders(customer_id);

A primary-key or UNIQUE constraint in PostgreSQL automatically creates a unique index; avoid building a duplicate index for exactly the same purpose.

## 2. Index scan versus sequential scan

- Sequential scan reads table pages and evaluates rows. It can be optimal for a small table or a predicate matching much of the table.
- Index scan navigates the index and may fetch matching table rows. It is useful for selective access but may incur many random heap fetches.
- Index-only scan can serve required columns from the index when the plan and PostgreSQL visibility information allow it.
- Bitmap scans collect matching row locations and then visit relevant heap pages; bitmap operations can combine index results.

Never treat every sequential scan as a bug. Ask how many rows and pages the query must read and which strategy has the lower cost.

## 3. Composite indexes

    CREATE INDEX idx_orders_customer_date
        ON orders(customer_id, created_at DESC);

This aligns with:

    SELECT order_id, created_at, total_amount
    FROM orders
    WHERE customer_id = 42001
    ORDER BY created_at DESC
    LIMIT 20;

B-tree ordering is based on the listed column sequence. The index can filter by customer and provide date ordering within the customer group.

For an index on (customer_id, status, created_at):
- Query filtering on customer_id can use the leading prefix.
- Query filtering on customer_id and status aligns with more of the key.
- Query filtering only by status or created_at usually cannot use the index as a direct leading-key lookup.
- Later-column access can sometimes be helped by specialized optimizer strategies, but do not design as though every column is independently indexed.

Column order depends on access patterns, equality versus range predicates, required ordering, prefix queries and write costs. “Most selective first” is not a universal rule.

## 4. Equality, range and ordering

A practical starting heuristic is leading equality filters, then useful range/order columns, then optional included columns.

    CREATE INDEX idx_orders_customer_date
        ON orders(customer_id, created_at);

    SELECT order_id, created_at
    FROM orders
    WHERE customer_id = 42001
      AND created_at >= TIMESTAMPTZ '2026-01-01 00:00:00+00'
    ORDER BY created_at;

Test the index against the actual query and data. The optimizer may still prefer another plan.

## 5. Selectivity

Selectivity describes how restrictive a predicate is. A predicate matching 100 rows from ten million is highly selective. Cardinality in a column often refers to its number of distinct values.

A unique order_id lookup is typically selective. A status predicate matching 40% of rows may not benefit from a standalone index if the query needs most of the table. Low-cardinality columns are not automatically useless: they can be useful as parts of composite indexes or partial-index predicates.

## 6. Partial indexes

    CREATE INDEX idx_pending_orders_created
        ON orders(created_at)
        WHERE status = 'PENDING';

This index stores entries only for pending rows. A query must imply the partial-index predicate in a way the planner can prove; parameterized queries can affect that proof. Frequent status changes can also cause index maintenance.

## 7. Expression indexes and sargability

A plain index on email may not be sufficient for a predicate on LOWER(email):

    CREATE INDEX idx_users_lower_email
        ON users(LOWER(email));

The query expression must align with the indexed expression. Define case normalization and uniqueness semantics according to the domain.

A date range predicate often works better with a regular timestamp index than wrapping the timestamp column in DATE(). Use correct timezone boundaries for the application's definition of a day.

Sargability describes whether a predicate can use an index efficiently to locate a search range. Function-wrapped columns and leading-wildcard LIKE patterns can prevent efficient use of an ordinary B-tree; specialized indexes may be appropriate for substring search.

## 8. Covering indexes

    CREATE INDEX idx_orders_customer_date_cover
        ON orders(customer_id, created_at DESC)
        INCLUDE (order_id, total_amount);

INCLUDE columns are payload, not additional ordering keys in PostgreSQL. This can enable an index-only scan for queries selecting those fields, depending on visibility-map state. Do not include every column: wide indexes consume storage, reduce cache efficiency and increase write work.

## 9. Specialized PostgreSQL index types

- B-tree: general equality, range and ordered access.
- GIN: inverted indexing such as arrays, JSONB containment and full-text search.
- GiST: extensible indexing for spatial, range and other operators.
- BRIN: small summaries of ranges across physical blocks; useful for very large data whose values correlate with row order.
- Hash: equality-oriented index; less general than B-tree.

Choose by operator and workload, not by memorizing the names.

## 10. Costs and workflow

Every index adds storage and relevant insert/update/delete maintenance. More indexes may reduce write throughput and increase cache pressure. Indexing a column that is never queried is wasted work.

Workflow:
1. Capture the real query, frequency and result size.
2. Understand data distribution and selectivity.
3. Propose the smallest index aligned with filtering/order.
4. Inspect EXPLAIN (ANALYZE, BUFFERS) safely.
5. Compare latency, buffers, write costs and regressions.
6. Remove redundant indexes only after validating dependencies and workload.

## References

- [PostgreSQL — Indexes](https://www.postgresql.org/docs/current/indexes.html)
- [PostgreSQL — Multicolumn indexes](https://www.postgresql.org/docs/current/indexes-multicolumn.html)
- [PostgreSQL — Indexes on expressions](https://www.postgresql.org/docs/current/indexes-expressional.html)
- [PostgreSQL — Partial indexes](https://www.postgresql.org/docs/current/indexes-partial.html)
- [PostgreSQL — Index-only scans and covering indexes](https://www.postgresql.org/docs/current/indexes-index-only-scans.html)
