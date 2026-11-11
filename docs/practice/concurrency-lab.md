# Concurrency Lab — Transactions and Race Conditions

These exercises are reasoning labs. Use an isolated local database and two separate sessions/connections. Do not run destructive concurrency experiments against production data.

## Scenario 1 — Lost update

Initial counter is 10. Two transactions both read 10; one adds 5 and the other adds 7 using application-side read/modify/write.

Tasks:
- Draw the interleaving.
- State the incorrect possible final value.
- Rewrite using an atomic UPDATE such as SET value = value + delta.
- Explain why row locking or a version check may also be relevant in a larger workflow.

## Scenario 2 — Last inventory item

Initial stock is 1. Two requests attempt to reserve one unit.

Use:

    UPDATE inventory
    SET stock = stock - 1
    WHERE product_id = 55
      AND stock >= 1;

Tasks:
- Run simultaneous attempts in two sessions.
- Check each affected-row count.
- Add a CHECK constraint for nonnegative stock.
- Explain why the API must treat zero updated rows as unsuccessful reservation.

## Scenario 3 — Deadlock

Transaction A locks account 1, then requests account 2. Transaction B locks account 2, then requests account 1.

Tasks:
- Reproduce the opposing lock order in a test environment.
- Observe which transaction the database aborts.
- Modify both workflows to acquire account locks in ascending account_id order.
- Add bounded retry handling where safe.

## Scenario 4 — Write skew

Two on-call doctors are represented by two rows. The rule is “at least one doctor remains on call.” Concurrent transactions both read that both doctors are on call, then each turns a different doctor off.

Tasks:
- Explain why updating different rows can still violate a cross-row invariant.
- Evaluate Serializable isolation with whole-transaction retries.
- Consider a shared guard row or schema redesign.
- Explain why row-locking only the row each transaction updates may be insufficient.

## Scenario 5 — Transaction and external side effect

A local transaction creates an order, calls a payment provider, then marks the order paid. The provider succeeds but the local transaction rolls back.

Tasks:
- Explain why database rollback cannot undo the remote charge.
- Propose durable payment/order states, idempotency, outbox/workflow delivery and reconciliation.
- Identify the authoritative status and recovery actions for each failure boundary.

## Scenario 6 — Serialization retries

Tasks:
- Define which errors are retriable in your client framework.
- Retry the whole transaction, not just the final statement, where earlier reads informed the decision.
- Bound attempts and use backoff as appropriate.
- Ensure external side effects are not repeated unsafely.

## Observation template

For each run, record:
- Database/version and isolation level.
- Transaction steps and timing.
- SQL statements and affected-row counts.
- Locks/wait events and error code.
- Expected versus observed state.
- Proposed fix and why it protects the invariant.
- Retest result.

## References

- [PostgreSQL — Transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [PostgreSQL — Explicit locking](https://www.postgresql.org/docs/current/explicit-locking.html)
- [Spring — Transaction management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)
