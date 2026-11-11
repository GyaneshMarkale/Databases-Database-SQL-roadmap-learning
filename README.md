# Database & SQL — Interview Preparation Roadmap

A structured, interview-focused curriculum for software engineers working with Java, Spring Boot, SQL databases, and distributed systems. The material follows seven modules from writing correct queries to choosing storage for a production system.

## Start here

1. [Module 1 — SQL Querying](docs/modules/module-01-sql-querying.md)
2. [Module 2 — Relational Modeling and Normalization](docs/modules/module-02-relational-modeling.md)
3. [Module 3 — Indexes and Query Performance](docs/modules/module-03-indexes.md)
4. [Module 4 — Transactions and Concurrency](docs/modules/module-04-transactions-concurrency.md)
5. [Module 5 — Execution Plans and Optimization](docs/modules/module-05-execution-plans.md)
6. [Module 6 — Spring Boot Data Access](docs/modules/module-06-spring-boot-data-access.md)
7. [Module 7 — Database and Storage Selection](docs/modules/module-07-storage-selection.md)

## Practice and revision

- [SQL interview exercises](docs/practice/sql-exercises.md)
- [Concurrency scenarios and lab exercises](docs/practice/concurrency-lab.md)
- [Interview readiness checklist](docs/interview-checklist.md)
- [Official documentation and learning resources](docs/reference-links.md)
- [Glossary](docs/glossary.md)
- [Query-tuning runbook template](docs/templates/query-tuning-runbook.md)
- [Schema design review template](docs/templates/schema-design-review.md)

## Learning approach

For every topic, be prepared to explain:
- The problem the concept solves.
- The correctness and performance trade-offs.
- A concrete SQL or Java example.
- Failure modes and what you would measure.
- Why an alternative might be better under different workload assumptions.

Use PostgreSQL as the baseline SQL dialect. Core concepts transfer to other relational databases, but syntax, execution plans, locking semantics, and isolation implementations can differ.

## Recommended workflow

- Read one module and reproduce the examples in a local PostgreSQL instance.
- Solve the exercises without looking at solutions first.
- Explain query plans using estimates and actual observations, not guesses.
- For Spring examples, record the SQL statements issued and verify transaction boundaries.
- Track weak areas in the checklist and revisit them in spaced revision.

## Scope and caveats

Examples are educational starting points, not production-tested financial or payment implementations. Before using patterns in production, validate them against your actual database version, schema, workload, security requirements, and failure model. Execution-plan timings are environment-dependent; do not treat illustrative numbers as benchmarks.
