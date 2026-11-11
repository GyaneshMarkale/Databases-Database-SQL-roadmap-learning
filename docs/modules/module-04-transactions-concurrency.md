# Module 4 — Transactions and Concurrency

## Outcomes

- Explain ACID and business invariants.
- Distinguish standard isolation levels and common anomalies.
- Use atomic conditional updates and row locks appropriately.
- Understand optimistic locking, deadlocks and transaction retries.
- Design safe retryable workflows.

## 1. ACID

- Atomicity: a transaction's changes commit together or roll back together.
- Consistency: each committed state satisfies database constraints and the application's required invariants.
- Isolation: concurrency behavior follows the selected isolation model.
- Durability: committed changes survive failures according to the database's configuration and recovery guarantees.

ACID does not make incorrect business logic correct. Add database constraints for invariants that can be represented there, and use concurrency control for multi-row or cross-operation rules.

## 2. Isolation levels and anomalies

Common anomalies:
- Dirty read: reading uncommitted data from another transaction.
- Non-repeatable read: rereading a row and seeing a different committed value.
- Phantom read: repeating a predicate query and getting a different set of rows.
- Lost update: one concurrent change overwrites another.
- Write skew: transactions read an overlapping snapshot and update different rows, jointly violating an invariant.

Standard SQL isolation guarantees are not identical to every engine's implementation. PostgreSQL treats READ UNCOMMITTED as READ COMMITTED. PostgreSQL REPEATABLE READ provides a stable snapshot and prevents phantoms in the usual SQL sense, but write skew can still occur. SERIALIZABLE detects unsafe executions and may abort transactions.

For PostgreSQL:
- READ COMMITTED: each statement sees a fresh snapshot of committed data as of its start.
- REPEATABLE READ: reads use a stable transaction snapshot.
- SERIALIZABLE: successful concurrent transactions are constrained to be equivalent to a serial execution, with possible serialization failures.

## 3. Atomic conditional updates

Dangerous pattern: read stock into application memory, calculate a new value, and later write it without protecting against concurrent updates.

Prefer an atomic conditional statement for the last-item reservation pattern:

    UPDATE inventory
    SET stock = stock - 1
    WHERE product_id = 55
      AND stock >= 1;

Inspect the affected-row count. One row updated means the condition was satisfied; zero rows means the product was missing or the reservation condition failed. Add CHECK (stock >= 0) as defense in depth if applicable.

For multi-item reservations, ensure all related updates share a transaction and failures roll back the whole reservation.

## 4. Pessimistic locking

    BEGIN;

    SELECT balance
    FROM accounts
    WHERE account_id = 101
    FOR UPDATE;

    -- Validate and update while the selected row is locked.

    COMMIT;

PostgreSQL SELECT FOR UPDATE acquires row-level locks on selected rows, blocking conflicting updates/locking operations until transaction end. Locking a query result does not automatically lock nonexistent rows that may match a future predicate.

Keep transactions short. Avoid remote calls while database locks and connections are held.

## 5. Optimistic locking

Add a version column and update conditionally:

    UPDATE products
    SET price = 1200,
        version = version + 1
    WHERE product_id = 55
      AND version = 7;

If no row is updated, the expected version may be stale or the row may have been deleted. Treat this as a conflict/missing record and follow the application's retry or conflict response policy. JPA supports version-based optimistic locking via @Version.

## 6. Deadlocks

A deadlock exists when transactions wait in a cycle. Example: T1 holds A and waits for B while T2 holds B and waits for A.

Mitigations:
- Acquire locks in a consistent order.
- Keep transactions short.
- Avoid unnecessary lock scope.
- Review access paths and long-running statements.
- Handle deadlock victims with bounded, safe retries.

PostgreSQL can abort one transaction to break the cycle. A retry must repeat the complete transactional decision when prior reads may be stale.

## 7. Transaction retries and idempotency

Serialization failures and deadlocks can be recoverable. Retry only when the operation is safe to repeat, bound attempts, and use backoff where appropriate. Do not duplicate external side effects during a retry.

An idempotency key or durable operation ID can prevent repeated requests from creating multiple logical transfers. A database transaction does not atomically include a remote payment provider: use explicit workflow states, idempotency, durable event delivery and reconciliation.

## 8. Transfer example checklist

For a transfer:
- Validate positive amount and authorization.
- Ensure source and destination exist.
- Debit atomically with sufficient-funds condition.
- Ensure the credit and debit participate in one database transaction.
- Verify affected row counts.
- Prevent duplicate logical transfers.
- Use consistent lock ordering if locking both accounts.
- Record audit/ledger data according to domain requirements.
- Design external provider interactions separately from local ACID boundaries.

## References

- [PostgreSQL — Transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [PostgreSQL — Explicit locking](https://www.postgresql.org/docs/current/explicit-locking.html)
- [PostgreSQL — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)
- [Spring Framework — Transaction management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)
