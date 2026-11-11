# Module 1 — SQL Querying and Problem Solving

## Outcomes

- Write joins without accidentally multiplying rows.
- Use GROUP BY, HAVING, subqueries, EXISTS and CTEs correctly.
- Solve top-N-per-group, ranking, running-total, and previous-row problems with window functions.
- Explain logical query processing and NULL-related traps.

## 1. Example schema and row grain

Use an e-commerce model:

- customers: one row per customer.
- orders: one row per order.
- order_items: one row per line item.
- products: one row per product.

Always identify the grain of each table before joining. Joining one customer to multiple orders creates multiple rows; joining those orders to their items can multiply rows again.

## 2. Joins

INNER JOIN returns matching pairs. LEFT JOIN preserves every left-side row and returns NULLs for unmatched right-side columns.

    SELECT c.customer_id, c.name, o.order_id
    FROM customers c
    LEFT JOIN orders o
      ON o.customer_id = c.customer_id;

To preserve all customers while attaching only completed orders, place the status predicate in the ON condition:

    SELECT c.customer_id, c.name, o.order_id
    FROM customers c
    LEFT JOIN orders o
      ON o.customer_id = c.customer_id
     AND o.status = 'COMPLETED';

Placing o.status = 'COMPLETED' in WHERE rejects rows where the joined order is NULL. That can turn the intended outer-join result into inner-join-like behavior.

### Anti-join: entities without matches

    SELECT c.customer_id, c.name
    FROM customers c
    WHERE NOT EXISTS (
        SELECT 1
        FROM orders o
        WHERE o.customer_id = c.customer_id
    );

NOT EXISTS is often a clear expression of “no related row.” Be careful with NOT IN when its subquery can return NULL, because SQL's three-valued logic can make the result unexpected.

## 3. Aggregations

    SELECT customer_id,
           COUNT(*) AS completed_orders,
           SUM(total_amount) AS total_spend,
           AVG(total_amount) AS average_order
    FROM orders
    WHERE status = 'COMPLETED'
    GROUP BY customer_id
    HAVING SUM(total_amount) > 50000
    ORDER BY total_spend DESC;

- WHERE filters rows before grouping.
- GROUP BY defines groups.
- HAVING filters groups using aggregate conditions.
- COUNT(*) counts rows; COUNT(column) ignores NULL values.
- COUNT(DISTINCT column) counts distinct non-NULL values.

Do not sum an order-level amount after joining to order_items unless you account for row multiplication. Aggregate at the correct grain, or use a separate pre-aggregation. SUM(DISTINCT amount) is not a safe general fix because different orders can share the same amount.

## 4. Subqueries and EXISTS

Scalar subquery — compare an order with the average amount:

    SELECT order_id, total_amount
    FROM orders
    WHERE total_amount > (
        SELECT AVG(total_amount)
        FROM orders
    );

Correlated EXISTS — find customers with a completed order:

    SELECT c.customer_id
    FROM customers c
    WHERE EXISTS (
        SELECT 1
        FROM orders o
        WHERE o.customer_id = c.customer_id
          AND o.status = 'COMPLETED'
    );

EXISTS expresses an existence test. The optimizer may transform it into a semi-join; do not assume the subquery must physically execute once per outer row.

## 5. Common table expressions

    WITH customer_spend AS (
        SELECT customer_id, SUM(total_amount) AS spend
        FROM orders
        WHERE status = 'COMPLETED'
        GROUP BY customer_id
    )
    SELECT c.name, s.spend
    FROM customer_spend s
    JOIN customers c ON c.customer_id = s.customer_id
    WHERE s.spend > 50000;

CTEs improve decomposition and readability. They are not automatically faster than subqueries; materialization and optimization behavior depend on the database and version.

## 6. Window functions

Window functions compute across related rows without collapsing the result to one row per group.

    SELECT order_id,
           customer_id,
           total_amount,
           SUM(total_amount) OVER (
               PARTITION BY customer_id
           ) AS customer_spend
    FROM orders;

Top three orders per customer:

    WITH ranked AS (
        SELECT order_id, customer_id, total_amount,
               ROW_NUMBER() OVER (
                   PARTITION BY customer_id
                   ORDER BY total_amount DESC, order_id
               ) AS rn
        FROM orders
    )
    SELECT *
    FROM ranked
    WHERE rn <= 3;

- ROW_NUMBER assigns a unique sequence number. Add a deterministic tie-breaker.
- RANK assigns equal ranks to ties and leaves gaps.
- DENSE_RANK assigns equal ranks but does not leave gaps.
- LAG and LEAD access previous or following rows in window order.
- Use an explicit ROWS frame for row-by-row running totals where peer rows should not share the same default frame behavior.

Running total:

    SELECT customer_id, order_id, created_at, total_amount,
           SUM(total_amount) OVER (
               PARTITION BY customer_id
               ORDER BY created_at, order_id
               ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
           ) AS running_total
    FROM orders;

A window function generally cannot be filtered in the WHERE clause at the same query level; use a CTE or subquery.

## 7. Logical processing model

A useful reasoning order is FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT. This is logical processing, not a promise of physical execution order; the optimizer may reorder operations.

## Practice prompts

1. Find customers with no orders.
2. Find the second-highest distinct salary.
3. Return the top three products by revenue in each category.
4. Find each customer's first and most recent order.
5. Compute a running balance per account.
6. Find duplicate email addresses.
7. Find customers who placed orders in two consecutive months.
8. Explain why an aggregate becomes incorrect after a one-to-many join.

## Self-review

Can you state the expected row grain before a join? Can you distinguish WHERE from HAVING? Can you explain ties in RANK versus DENSE_RANK? Can you identify NULL-related pitfalls? If not, repeat the relevant example with a small hand-worked dataset.

## References

- [PostgreSQL — Queries](https://www.postgresql.org/docs/current/queries.html)
- [PostgreSQL — Table expressions and joins](https://www.postgresql.org/docs/current/queries-table-expressions.html)
- [PostgreSQL — Window functions](https://www.postgresql.org/docs/current/functions-window.html)
- [PostgreSQL — Aggregate functions](https://www.postgresql.org/docs/current/functions-aggregate.html)
- [PostgreSQL Exercises](https://pgexercises.com/)
- [LeetCode Database problems](https://leetcode.com/problemset/database/)
