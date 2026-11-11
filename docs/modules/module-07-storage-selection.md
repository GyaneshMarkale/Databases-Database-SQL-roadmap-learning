# Module 7 — Database Selection and Storage Trade-offs

## Outcomes

- Choose a store based on access patterns, guarantees, scale and operational constraints.
- Compare relational, document, key-value, wide-column, search and analytical systems.
- Explain consistency and partitioning trade-offs without overgeneralizing SQL versus NoSQL.
- Design a small polyglot-persistence architecture and identify its failure modes.

## 1. Start from requirements

Before choosing a product, write down:
1. Access patterns: point reads, range queries, joins, text search, aggregates, writes.
2. Correctness: invariants, uniqueness, transaction scope, staleness tolerance and durability.
3. Scale: data volume, throughput, peak load, latency and growth.
4. Distribution: partitioning, hot keys, replication and multi-region needs.
5. Operations: cost, backups, restores, monitoring, migrations, staffing and recovery.

A database category alone does not tell you the consistency or scalability guarantees. Evaluate the actual engine, configuration and operations.

## 2. Relational systems

Examples: PostgreSQL, MySQL, SQL Server and distributed SQL products.

Strengths:
- joins and flexible queries,
- integrity constraints,
- transactions over related rows,
- useful relational model for orders, accounts, payments and ledgers.

Trade-offs:
- query/index tuning and connection management still matter,
- distributed transactions and cross-shard queries can be harder,
- replicas, partitions or sharding introduce operational considerations.

Start with relational storage for transactional relational data unless a concrete requirement indicates otherwise.

## 3. Document stores

MongoDB is an example. Documents can embed related data or reference other documents. Embedding can reduce read round trips and let related data be updated as one document; referencing can fit independently queried or unbounded data.

Trade-offs include document growth, duplicated facts, consistency across documents, validation and indexing. Flexible structure does not mean schema design is unnecessary. Document stores can support transactions; choose based on the actual semantics and access pattern.

## 4. Key-value stores and caches

Redis provides strings, hashes, sets, sorted sets, lists, streams and other data structures. Use cases include caching, sessions, rate limits and operational data structures. Redis persistence options vary; decide whether it is a cache, durable store or authoritative system based on the required recovery guarantees.

DynamoDB is a managed key-value/document database. Its data model depends on partition keys, optional sort keys and indexes. Design around known access patterns, avoid hot keys and understand read-consistency choices. Do not expect arbitrary relational joins to be the normal access model.

## 5. Wide-column stores

Cassandra is designed for distributed workloads modeled around queries, partition keys and clustering columns. It often uses denormalized tables so a known query is served efficiently. Poor partition-key choice can create hot partitions or oversized partitions. It is not a drop-in replacement for a relational database whose workload depends heavily on joins and cross-entity constraints.

## 6. Search systems

Elasticsearch and OpenSearch are useful for full-text search, relevance ranking, filtering and faceting. They usually complement a system of record. An asynchronous indexing pipeline can lag, fail, retry or duplicate events; the design needs a source of truth, reindex strategy and stale-search expectations.

Do not use a search index as the authority for payment or inventory correctness merely because it is fast at retrieval.

## 7. Analytical stores and time series

Analytical engines such as ClickHouse are designed for scan-heavy aggregation and observability/warehouse patterns. Separate OLAP storage can isolate reports from latency-sensitive OLTP transactions, but adds ingestion delay and operational cost.

Time-series workloads need efficient time filtering, retention, aggregation and often downsampling. A dedicated time-series store is useful when its features justify the complexity; PostgreSQL may be sufficient for a smaller workload.

## 8. Consistency and CAP

Do not say “SQL is CP” or “NoSQL is AP.” During a network partition, a distributed system cannot guarantee both linearizable consistency and availability for every request. Product behavior depends on the consistency model, replication scheme, configuration and operation.

Ask separately:
- Can the read be stale?
- Does an acknowledged write survive the required failures?
- Does the operation span multiple rows/documents/partitions?
- What does the system do during partition or replica lag?
- Can the application retry safely?

Examples:
- Inventory reservation: enforce correctness in an authoritative transactional write path.
- Product search: a short index delay may be acceptable.
- Analytics: eventual ingestion may be acceptable if the lag is understood and visible.
- Session cache: decide TTL, invalidation and recovery behavior explicitly.

## 9. E-commerce architecture example

Possible starting layout:
- PostgreSQL: orders, inventory and transactional records.
- Redis: hot cache/session/rate-limit use cases, only when measured benefit justifies it.
- Search engine: product relevance and text search.
- Analytical store: event aggregations and dashboards.

A durable event/CDC pipeline can propagate changes to search and analytics. Define retries, idempotency, ordering, dead-letter handling, rebuild/reconciliation and lag monitoring.

Do not introduce every technology by default. Each store increases operational, security, observability, recovery and consistency responsibilities. For a smaller workload, one well-designed relational database can be the better choice.

## 10. Interview decision framework

1. Describe the query/write patterns and latency objectives.
2. State invariants, consistency and transaction requirements.
3. Estimate size and peak throughput.
4. Evaluate partitioning, hot keys and failure recovery.
5. Compare operational cost and team expertise.
6. Validate with realistic load and failure tests.
7. Explain alternatives rejected and what evidence would change the decision.

## References

- [MongoDB — Data modeling](https://www.mongodb.com/docs/manual/data-modeling/)
- [DynamoDB — Core components](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
- [DynamoDB — Read consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
- [Apache Cassandra — Data modeling introduction](https://cassandra.apache.org/doc/latest/cassandra/developing/data-modeling/intro.html)
- [Redis — Data types](https://redis.io/docs/latest/develop/data-types/)
- [Redis — Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
- [Elasticsearch — Full-text search](https://www.elastic.co/docs/solutions/search/full-text)
- [ClickHouse — Use cases](https://clickhouse.com/use-cases)
