# Module 6 — Spring Boot Data Access

## Outcomes

- Explain HikariCP and connection-pool sizing.
- Set safe Spring transaction boundaries.
- Recognize proxy/self-invocation and rollback pitfalls.
- Avoid N+1 queries and expensive deep pagination.
- Understand JDBC batching, flush/clear and persistence-context behavior.

## 1. Connection pooling with HikariCP

Spring Boot commonly uses HikariCP as its JDBC connection pool. A pool reuses a bounded set of connections; it does not make slow SQL faster.

    spring:
      datasource:
        url: jdbc:postgresql://localhost:5432/shop
        username: ${DB_USERNAME}
        password: ${DB_PASSWORD}
        hikari:
          maximum-pool-size: 20
          minimum-idle: 5
          connection-timeout: 3000
          idle-timeout: 600000
          max-lifetime: 1800000

Values are illustrative and must be tuned for the actual workload. connection-timeout is the wait for a pool connection, not a SQL execution timeout.

Across eight replicas with a pool maximum of 20 each, the service can potentially use 160 connections, before accounting for other clients. Pool sizing must consider the database's practical capacity, replica count, transaction duration, other workloads and measured waits. Increasing pool size can reduce waiting initially but can also increase database contention.

Little's Law: average occupied resources L ≈ throughput λ × average occupancy time W. If 100 operations/sec each occupy a connection for 0.1 sec, the average occupancy is approximately 10 connections; this is not a safe maximum-pool recommendation.

## 2. Transaction boundaries

Put a business operation's transaction at a service boundary when multiple database operations must succeed or fail together.

    @Service
    public class OrderService {
        private final OrderRepository orderRepository;
        private final OrderItemRepository itemRepository;

        @Transactional
        public void createOrder(Order order, List<OrderItem> items) {
            orderRepository.save(order);
            itemRepository.saveAll(items);
        }
    }

This is a conceptual example; entity relationships, cascade behavior, flush timing, validation and idempotency must match the actual domain.

### Proxy and rollback traps

- With common proxy-based Spring transactions, a method calling another method on the same object can bypass the proxy. Do not assume the inner annotation creates a separate transaction.
- By default, unchecked RuntimeException and Error trigger rollback; checked exceptions do not automatically trigger rollback in every configuration. Set rollback rules explicitly when required.
- REQUIRED joins an existing transaction or starts one if absent.
- REQUIRES_NEW suspends an existing transaction and starts a separate one; it can commit despite the outer transaction later rolling back and may require another connection.
- readOnly=true is a hint/optimization in common setups, not a universal security boundary that makes every write impossible.
- A database transaction cannot roll back a completed remote HTTP/payment side effect.

Keep external calls outside long-running database transactions where possible. Use an explicit workflow, idempotency and reconciliation for cross-system operations.

## 3. N+1 queries

A query loads N orders and then application code reads each order's lazy customer. The ORM can issue extra queries for related customers. The actual count depends on mappings, cache and distinct related entities.

Solutions:
- fetch join for appropriate to-one relationships,
- entity graphs,
- DTO projections for read-only endpoints,
- batch fetching when loading in groups is suitable.

Do not switch everything to EAGER. Multiple to-many fetch joins can multiply rows and complicate pagination.

## 4. DTO projection

If the endpoint only needs an order ID, customer name and amount, query those columns rather than materializing every entity and relationship. Projection reduces transferred columns and ORM work for suitable read paths.

## 5. Pagination

Offset example:

    SELECT order_id, created_at
    FROM orders
    WHERE customer_id = 42001
    ORDER BY created_at DESC, order_id DESC
    LIMIT 20 OFFSET 10000;

Deep offsets may require walking past many rows. Spring Data Page can also execute a count query; a Slice-style result can avoid requiring total-element counts in suitable methods.

Keyset/seek pagination:

    SELECT order_id, created_at
    FROM orders
    WHERE customer_id = 42001
      AND (created_at, order_id)
          < (TIMESTAMPTZ '2026-11-10 10:00:00+00', 9021)
    ORDER BY created_at DESC, order_id DESC
    LIMIT 20;

Use a deterministic order and a matching index. Handle NULL sort values and define semantics for rows changing during pagination.

## 6. JDBC batching and Hibernate

Example Hibernate properties:

    spring:
      jpa:
        properties:
          hibernate:
            jdbc:
              batch_size: 50
            order_inserts: true
            order_updates: true

Driver behavior, identifier strategy, statement compatibility and provider settings affect whether batching is actually used. Verify it rather than assuming the configuration guarantees it.

For large entity writes, periodically flush and clear the persistence context:

    for (int i = 0; i < items.size(); i++) {
        entityManager.persist(items.get(i));
        if ((i + 1) % 50 == 0) {
            entityManager.flush();
            entityManager.clear();
        }
    }

flush sends pending changes to the database but does not commit. clear detaches entities. Clearing before flushing pending changes can produce unexpected behavior. Transaction chunk size, JDBC batch size and flush/clear interval solve different concerns and need not be equal.

Bulk JPQL DML avoids loading every entity but bypasses ordinary per-entity dirty checking and lifecycle callbacks; managed entities can become stale. Consider flush/clear or refresh behavior.

## 7. Monitoring checklist

Track:
- active, idle and pending pool connections;
- connection acquisition timeout/count and wait duration;
- SQL latency and query frequency;
- transaction duration and rollback count;
- slow query logs and execution plans;
- N+1/query-count regressions;
- lock waits, deadlocks and database resource use.

## References

- [Spring Boot — SQL databases](https://docs.spring.io/spring-boot/reference/data/sql.html)
- [Spring Framework — Transaction management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)
- [Spring Data JPA reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Spring Data JPA — Query methods](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)
- [Hibernate ORM User Guide](https://docs.hibernate.org/orm/current/userguide/html_single/)
- [HikariCP](https://github.com/brettwooldridge/HikariCP)
