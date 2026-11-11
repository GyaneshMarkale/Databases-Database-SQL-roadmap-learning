# Official and High-Value Learning Resources

Use product documentation as the source of truth for engine-specific behavior. Check the version matching the database or framework you actually use.

## PostgreSQL — SQL, schema, indexes and execution plans

- [Tutorial](https://www.postgresql.org/docs/current/tutorial.html) — starting point for SQL and transactions.
- [Queries](https://www.postgresql.org/docs/current/queries.html) — query expressions and processing.
- [Table expressions and joins](https://www.postgresql.org/docs/current/queries-table-expressions.html) — join behavior.
- [Window functions](https://www.postgresql.org/docs/current/functions-window.html) — ranking and window calculations.
- [Aggregate functions](https://www.postgresql.org/docs/current/functions-aggregate.html) — aggregation semantics.
- [Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) — PK, FK, UNIQUE, CHECK and exclusion constraints.
- [Indexes overview](https://www.postgresql.org/docs/current/indexes.html) — index concepts.
- [Multicolumn indexes](https://www.postgresql.org/docs/current/indexes-multicolumn.html) — composite index behavior.
- [Indexes on expressions](https://www.postgresql.org/docs/current/indexes-expressional.html) — expression indexes.
- [Partial indexes](https://www.postgresql.org/docs/current/indexes-partial.html) — predicate-based indexes.
- [Index-only scans and covering indexes](https://www.postgresql.org/docs/current/indexes-index-only-scans.html) — included columns and visibility.
- [EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html) — reading query plans.
- [EXPLAIN command reference](https://www.postgresql.org/docs/current/sql-explain.html) — supported options and caveats.
- [Planner statistics](https://www.postgresql.org/docs/current/planner-stats.html) — statistics and estimates.
- [Transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html) — isolation behavior.
- [Explicit locking](https://www.postgresql.org/docs/current/explicit-locking.html) — locks and conflict behavior.
- [Transaction tutorial](https://www.postgresql.org/docs/current/tutorial-transactions.html) — BEGIN/COMMIT/ROLLBACK.
- [Routine vacuuming and statistics maintenance](https://www.postgresql.org/docs/current/routine-vacuuming.html) — maintenance concepts.

## SQL practice

- [PostgreSQL Exercises](https://pgexercises.com/) — hands-on SQL problem sets.
- [LeetCode Database problems](https://leetcode.com/problemset/database/) — interview-style questions.
- [SQLBolt](https://sqlbolt.com/) — interactive introductory SQL.
- [IBM — Database normalization](https://www.ibm.com/think/topics/database-normalization) — normalization concepts.

## Spring Boot, Spring Data and Hibernate

- [Spring Boot — SQL databases](https://docs.spring.io/spring-boot/reference/data/sql.html) — data source and SQL support.
- [Spring Framework — Transaction management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html) — declarative and programmatic transactions.
- [Spring Data JPA reference](https://docs.spring.io/spring-data/jpa/reference/) — repositories, query methods and persistence.
- [Spring Data JPA — Query methods](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html) — query declaration options.
- [Hibernate ORM User Guide](https://docs.hibernate.org/orm/current/userguide/html_single/) — fetching, batching and persistence-context behavior.
- [HikariCP](https://github.com/brettwooldridge/HikariCP) — configuration and connection-pool behavior.

## Document and key-value stores

- [MongoDB — Data modeling](https://www.mongodb.com/docs/manual/data-modeling/) — embed versus reference and workload modeling.
- [MongoDB — Embedded data](https://www.mongodb.com/docs/manual/data-modeling/embedding/) — use cases and trade-offs of embedding.
- [DynamoDB — Core components](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html) — partition keys and sort keys.
- [DynamoDB — Read consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html) — read guarantees by operation/index.
- [Redis — Data types](https://redis.io/docs/latest/develop/data-types/) — strings, hashes, sets, sorted sets and streams.
- [Redis — Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) — RDB, AOF and persistence options.

## Wide-column, search and analytics

- [Apache Cassandra — Data modeling introduction](https://cassandra.apache.org/doc/latest/cassandra/developing/data-modeling/intro.html) — query-first schema design.
- [Apache Cassandra — CQL data definition](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/ddl.html) — partition and clustering columns.
- [Elasticsearch — Full-text search](https://www.elastic.co/docs/solutions/search/full-text) — text analysis and relevance.
- [ClickHouse — Use cases](https://clickhouse.com/use-cases) — analytics, observability and warehousing.

## How to use the resource list

1. Read the module notes first to form a mental model.
2. Open the official reference for behavior that is engine/framework-specific.
3. Reproduce the relevant behavior in a local or disposable environment.
4. Record the database/framework version and observed result.
5. Never rely on an interview summary when correctness depends on a version-specific detail.
