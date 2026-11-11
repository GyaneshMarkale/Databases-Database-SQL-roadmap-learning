# SQL Interview Practice Set

Use PostgreSQL syntax. Unless stated otherwise, assume an e-commerce schema with customers, orders, order_items and products. Write the query first, then explain its row grain, NULL handling, correctness and potential index needs.

## A. Joins and aggregation

1. List every customer and their most recent order if one exists.
2. Find customers who have never placed an order.
3. Return completed-order spend per customer, including customers with zero completed orders.
4. Find categories with revenue above a threshold.
5. Detect products with no order items.
6. Explain and fix inflated order totals when orders join to multiple order_items.
7. Find customers with at least three orders in the last 90 days.
8. Return monthly revenue and month-over-month change.

## B. Subqueries and CTEs

9. Find orders above the overall average order amount.
10. Find customers who placed at least one completed order using EXISTS.
11. Find products whose list price is higher than the average in their category.
12. Use a CTE to calculate spend per customer and return the top spenders.
13. Explain why NOT IN can behave unexpectedly when the subquery contains NULL.

## C. Window functions

14. Return the top three products by revenue per category. Decide whether ties should be included.
15. Return the latest order per customer.
16. Compute a running total of completed spend per customer.
17. Compare each customer's order amount with the previous order using LAG.
18. Assign RANK and DENSE_RANK to employees with tied salaries and explain the gaps.
19. Find each customer's first order and days until their second order.
20. Find the largest gap between consecutive events for each device.

## D. Data quality

21. Find duplicate email addresses under the application's normalization rules.
22. Detect order items with quantities that violate a business rule.
23. Find orders whose stored total disagrees with the recomputed line total. Define whether taxes/discounts/shipping are included before writing the query.
24. Find orphan references if the schema lacks an enforced foreign key.
25. Identify orders that have multiple successful payment records when that violates the domain model.

## E. Performance and correctness review

26. Given an execution plan, identify the node with the largest actual row-processing cost.
27. Propose an index for customer history ordered by created_at descending. Explain the leading columns.
28. Rewrite a date equality predicate into a timestamp range, preserving the intended timezone boundaries.
29. Compare OFFSET pagination with keyset pagination for a deep result page.
30. Explain why a scalar aggregate can become inflated after a one-to-many join.

## Expected answer format

For each exercise, document:
- SQL query.
- Assumptions and edge cases.
- Expected row grain and result cardinality.
- NULL/tie behavior.
- Potential index or execution-plan concerns.
- At least one test dataset that proves the result.

## Practice references

- [PostgreSQL tutorial](https://www.postgresql.org/docs/current/tutorial.html)
- [PostgreSQL Exercises](https://pgexercises.com/)
- [LeetCode Database problem set](https://leetcode.com/problemset/database/)
- [SQLBolt interactive lessons](https://sqlbolt.com/)
