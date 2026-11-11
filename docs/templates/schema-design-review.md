# Relational Schema Design Review Template

## 1. Domain and requirements

- Business entities:
- Core operations:
- Reads and writes:
- Volume and peak throughput:
- Consistency and durability requirements:
- Retention, audit and privacy requirements:

## 2. Entity and row grain

For each table:
- What does one row represent?
- Which attributes identify it?
- Which fields are required/optional?
- Which facts are historical snapshots?
- What are the lifecycle and deletion rules?

## 3. Keys and constraints

- Primary and candidate keys:
- Natural versus surrogate key decisions:
- Composite keys:
- Unique constraints:
- Foreign keys and nullability:
- CHECK constraints:
- Referential actions:
- Uniqueness semantics involving case, normalization and NULL:

## 4. Relationships

- 1:1 relationships and enforcement:
- 1:N relationships and FK placement:
- M:N junction tables and their keys:
- Optionality:
- Unbounded child collections or data growth:

## 5. Normalization and duplication

- Functional dependencies:
- 1NF/2NF/3NF/BCNF considerations:
- Update/insert/delete anomalies:
- Required denormalization or snapshots:
- Source of truth and consistency/reconciliation process:

## 6. Access patterns and indexes

For each important query:
- Exact predicates:
- Sort and pagination:
- Expected result cardinality:
- Candidate index and column order:
- Selectivity/distribution assumptions:
- Read versus write and storage trade-offs:
- Execution-plan validation approach:

## 7. Transactions and concurrency

- Invariants that must never be violated:
- Transaction boundary:
- Isolation level:
- Atomic conditional updates or locks:
- Optimistic version checks:
- Retry/idempotency strategy:
- Deadlock risks and lock ordering:
- External side effects and recovery:

## 8. Operations

- Connection budget and pool sizing:
- Migration/backfill approach:
- Backup and restore:
- Monitoring and alerts:
- Failure recovery:
- Cost and ownership:
- Alternatives considered and reasons rejected:
