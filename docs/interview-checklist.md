# Database & SQL Interview Readiness Checklist

Use this as a self-assessment. Mark each item only when you can explain it and work through an example without relying on notes.

## Module 1 — SQL
- [ ] INNER vs LEFT JOIN and predicates in ON versus WHERE.
- [ ] Explain row multiplication across 1:N joins.
- [ ] GROUP BY, HAVING, COUNT(*) versus COUNT(column).
- [ ] EXISTS, NOT EXISTS, IN, and NULL issues with NOT IN.
- [ ] CTEs and logical query processing.
- [ ] ROW_NUMBER, RANK and DENSE_RANK.
- [ ] Running totals, LAG/LEAD and top-N-per-group.

## Module 2 — Modeling
- [ ] Primary, candidate, natural, surrogate and composite keys.
- [ ] Foreign keys, uniqueness, nullability, CHECK constraints.
- [ ] Functional dependencies and normalization anomalies.
- [ ] 1NF, 2NF, 3NF and BCNF.
- [ ] Model 1:1, 1:N and M:N with optionality.
- [ ] Choose CASCADE/RESTRICT/SET NULL intentionally.
- [ ] Explain when transaction snapshots or denormalized aggregates are justified.

## Module 3 — Indexes
- [ ] B-tree lookup, range access and ordered retrieval.
- [ ] Index scan versus sequential scan and why the planner chooses each.
- [ ] Composite index ordering and leading prefixes.
- [ ] Selectivity, cardinality and skew.
- [ ] Partial, expression and covering indexes.
- [ ] Index write/storage costs and duplicate index detection.

## Module 4 — Concurrency
- [ ] ACID and business invariants.
- [ ] Dirty/non-repeatable/phantom reads, lost update and write skew.
- [ ] PostgreSQL isolation behavior versus the standard model.
- [ ] Atomic conditional updates and affected-row checks.
- [ ] Pessimistic and optimistic locking.
- [ ] Deadlock causes, consistent lock order and safe retries.
- [ ] Idempotency and boundaries of a local database transaction.

## Module 5 — Execution plans
- [ ] EXPLAIN versus EXPLAIN ANALYZE; remember ANALYZE executes the statement.
- [ ] Startup/total cost versus elapsed time.
- [ ] Estimated versus actual rows, loops and row-estimation errors.
- [ ] Sequential, index, index-only and bitmap scans.
- [ ] Nested-loop, hash and merge joins.
- [ ] Sort spills, temporary I/O and buffer interpretation.
- [ ] Use a before/after query-tuning workflow.

## Module 6 — Spring Boot
- [ ] HikariCP pool and acquisition timeout vs query timeout.
- [ ] Budget connections across replicas and other clients.
- [ ] @Transactional service boundary and proxy/self-invocation behavior.
- [ ] REQUIRED versus REQUIRES_NEW and rollback rules.
- [ ] N+1 diagnosis and DTO/fetch/batch strategies.
- [ ] Page versus Slice and offset versus keyset pagination.
- [ ] JDBC batch, flush/clear and transaction chunking.
- [ ] Monitor pool wait, SQL latency, transaction duration and lock waits.

## Module 7 — Storage choices
- [ ] Relational versus document versus key-value versus wide-column models.
- [ ] Search/analytics/time-series workload differences.
- [ ] Partition keys, hot keys, access patterns and denormalization.
- [ ] Consistency and replication semantics of the actual product.
- [ ] CAP in the context of a network partition.
- [ ] Polyglot persistence costs and source-of-truth ownership.
- [ ] Defend the store choice using workload, correctness, scale and operations.

## Final design exercise

Design a checkout flow that reserves inventory, creates an order and integrates with a remote payment provider. Explain:
- schema and constraints,
- transaction boundary,
- concurrent reservation behavior,
- idempotency,
- external call failure handling,
- query indexes and execution plans,
- connection pool sizing,
- event delivery and reconciliation,
- operational metrics and recovery.

A strong answer includes trade-offs, failure modes and verification steps—not only a diagram or technology list.
