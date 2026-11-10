# SQL result semantics

- Establish the grain of every input/result. One-to-many joins can multiply amounts before aggregation; preaggregate at the required grain or use EXISTS for existence, not a compensating DISTINCT that hides errors.
- Predicates on the nullable side in WHERE can turn a LEFT JOIN into an inner join. Put predicates in ON when unmatched left rows must survive.
- COUNT(*) counts rows; COUNT(column) counts non-NULL values. SUM over no rows is NULL. COALESCE to zero only when domain semantics require it.
- NOT IN can become unknown when its right side contains NULL. Prefer a correctly correlated NOT EXISTS when that matches intended null/key semantics. Use IS NULL or IS [NOT] DISTINCT FROM for the required NULL comparison.
- CTEs express structure, not guaranteed materialization or speed. Inlining/materialization depends on PostgreSQL version, use count, volatility and explicit options. Preserve the meaning of volatile functions and data-modifying CTEs.
- ORDER BY must include a tie-breaker where deterministic row selection/pagination matters. Keyset predicates must match sort directions and NULL ordering. Row comparisons with NULL need special handling.
- ROW_NUMBER breaks ties; RANK retains peers. The default ordered window frame can include peers; use an explicit ROWS frame for a row-by-row running total or an appropriate full-partition frame for last_value.
- Compare rewritten results with EXCEPT ALL in both directions to retain duplicate multiplicity. This compares bags, not order, type metadata or external side effects; check those separately.

Self-contained example: retain customers without paid orders and avoid multiplying amounts.

```sql
WITH customers(id) AS (VALUES (1), (2)),
orders(customer_id, amount, status) AS (
  VALUES (1, 10::numeric, 'paid'), (1, 20::numeric, 'pending')
)
SELECT c.id, COALESCE(SUM(o.amount), 0) AS paid_total
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id AND o.status = 'paid'
GROUP BY c.id
ORDER BY c.id;
```

Self-contained example: deterministic running total, including tied timestamps.

```sql
WITH events(id, occurred_at, amount) AS (
  VALUES (1, TIMESTAMPTZ '2026-01-01 00:00:00+00', 10::numeric),
         (2, TIMESTAMPTZ '2026-01-01 00:00:00+00', 20::numeric)
)
SELECT id, SUM(amount) OVER (
  ORDER BY occurred_at, id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS running_total
FROM events
ORDER BY occurred_at, id;
```

Sources: [Table expressions](https://www.postgresql.org/docs/current/queries-table-expressions.html), [windows](https://www.postgresql.org/docs/current/functions-window.html), [subqueries](https://www.postgresql.org/docs/current/functions-subquery.html).
