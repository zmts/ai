# PostgreSQL — Normalization, Denormalization, and JOINs

## 1. Normalization: Separate Facts by Meaning

Normalization reduces redundancy and update anomalies: the same fact does not
have to be corrected in many rows.

Example model:

- `users(id, name)` — a user.
- `orders(id, user_id, status)` — an order referencing a user.
- `order_items(id, order_id, product_id, quantity, unit_price)` — order line items.

The user's current name lives in `users` rather than being copied into every order.
Foreign keys enforce relationships; the corresponding constraints enforce
required values and uniqueness.

A brief guide to normal forms:

| Form | Practical guideline |
|---|---|
| 1NF | No repeating groups such as `product_1`, `product_2`; line items are represented as rows |
| 2NF | Non-key attributes depend on the entire candidate key, not part of it |
| 3NF | No transitive dependency of non-key attributes on a key through other non-key attributes |

These are brief guidelines, not complete formal definitions.

**When to use:** data changes frequently, and integrity, consistency, and
independent evolution of entities matter.
This is a reasonable starting point for an ordinary application database.
There is no need to put every field in a separate table.

## 2. Denormalization: Store Data for a Specific Read

Denormalization deliberately adds copies or derived data:
for example, `orders.items_total` or a table of daily totals.

| Situation | Approach |
|---|---|
| Ordinary CRUD, with queries meeting requirements | Keep the normalized model |
| A frequent expensive report after checking queries and indexes | Consider an aggregate table or materialized view |
| Fast access to a computed value is needed | Store the result with a clear update mechanism |
| Acceptable lag and synchronization are undefined | Define them first, then denormalize |

For each stored result, define:
**source of truth → update owner → acceptable staleness → recomputation process**.
Synchronous updates can share a transaction with the source-data change;
asynchronous updates need handling for delays and retries.

An ordinary `VIEW` stores a query; a `MATERIALIZED VIEW` stores its result,
which must be updated with `REFRESH`.

**A historical snapshot is a separate fact.** The purchase price in
`order_items.unit_price` does not have to match the product's current price.
Do not automatically “synchronize” it when the catalog changes.

## 3. Choosing a JOIN

Choose a JOIN for the required result, not its presumed speed.

| Type | Result | When to use |
|---|---|---|
| `INNER JOIN` | Matching pairs only | Orders together with existing users |
| `LEFT JOIN` | All left rows and right matches; otherwise right-side `NULL` | All users, including those without orders |
| `RIGHT JOIN` | All right rows and left matches | Mirror of LEFT; swapping the tables is often clearer |
| `FULL OUTER JOIN` | Matches and unmatched rows from both sides | Reconciling two sets |
| `CROSS JOIN` | Every combination: N × M rows | A “product × region” matrix |
| Self join | A table joined to itself through different aliases | Employee and manager; a usage of JOIN, not a separate JOIN type |

### INNER JOIN: Orders with User Names

```sql
SELECT o.id, u.name
FROM orders AS o
JOIN users AS u ON u.id = o.user_id;
```

### LEFT JOIN: All Users and Their Paid Orders

```sql
SELECT u.id, o.id AS order_id
FROM users AS u
LEFT JOIN orders AS o
    ON o.user_id = u.id
   AND o.status = 'paid';
```

The status predicate in `ON` preserves users without paid orders.
Moving `o.status = 'paid'` into `WHERE` excludes them.

### EXISTS: Users with Orders, Not the Orders Themselves

```sql
SELECT u.id
FROM users AS u
WHERE EXISTS (
    SELECT 1 FROM orders AS o WHERE o.user_id = u.id
);
```

`EXISTS` does not repeat each user for every matching order.
Use `NOT EXISTS` for users without orders.

## 4. Common Mistakes

- **Row multiplication.** One order with three items produces three joined
  rows. Adding two payments can produce six. Before summing, aggregate
  independent child sets separately to the required grain, such as one row
  per order.
- **DISTINCT hides the cause.** It does not correct an already inflated sum.
- **COUNT after LEFT JOIN.** `COUNT(*)` includes the unmatched row;
  use `COUNT(o.id)` for the number of orders if `id` cannot be NULL.
- **NULL is not compared with `= NULL`.** Use `IS NULL`.
  For absence of related rows, `NOT EXISTS` is often clearer than `NOT IN`,
  whose behavior is sensitive to NULL.
- **NATURAL JOIN depends on column names.** Prefer explicit `ON` or `USING`
  so adding a same-named column does not change the relationship.

## 5. What to Do When a JOIN Is Slow

First check join conditions, row counts, statistics, indexes, and the actual
plan. A large table count alone does not justify denormalization.

`Nested Loop`, `Hash Join`, and `Merge Join` are execution algorithms;
`INNER`/`LEFT`/`FULL` define result semantics. The planner chooses the
algorithm; none is universally best.

See also: [indexes and EXPLAIN](postgresql-indexes-cheatsheet.md#9-checking-an-index-with-explain),
[transactions](postgresql-transactions-cheatsheet.md).

### Sources

- [Normalization basics — Microsoft](https://learn.microsoft.com/en-us/previous-versions/troubleshoot/microsoft-365/microsoft-365-apps/access/database-normalization-description)
- [JOINs and table expressions — PostgreSQL](https://www.postgresql.org/docs/current/queries-table-expressions.html)
- [EXISTS and NOT IN — PostgreSQL](https://www.postgresql.org/docs/current/functions-subquery.html)
- [Materialized Views — PostgreSQL](https://www.postgresql.org/docs/current/rules-materializedviews.html)
- [EXPLAIN — PostgreSQL](https://www.postgresql.org/docs/current/using-explain.html)
