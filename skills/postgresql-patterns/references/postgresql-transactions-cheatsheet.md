# PostgreSQL Transactions — Cheatsheet

## 1. What a Transaction Guarantees

A **transaction** groups related data operations: they commit together or roll back.

| ACID property | Practical meaning |
|---|---|
| Atomicity | A partially completed operation must not remain in the database |
| Consistency | Database constraints and correct application logic preserve data invariants |
| Isolation | Visibility and interaction between concurrent transactions depend on the isolation level |
| Durability | Committed changes survive a failure; guarantees depend on durability settings |

**Atomicity does not eliminate races:** two clients can read the same data and make conflicting decisions. See the [locking cheatsheet](postgresql-locks-cheatsheet.md#3-mvcc-and-isolation-levels) for isolation and concurrency protection.

## 2. Boundaries: BEGIN / COMMIT / ROLLBACK

- Without an explicit transaction block, every statement runs in a transaction; under ordinary autocommit, a successful statement commits automatically. The driver may manage these boundaries itself.
- `BEGIN` starts an explicit transaction.
- `COMMIT` commits changes; an error can occur here too, for example during a deferred constraint check.
- `ROLLBACK` undoes changes in the current transaction.

### Example: An Order and Its Items

This assumes existing `orders` and `order_items` tables; the database generates `orders.id`.

```sql
BEGIN;

WITH new_order AS (
    INSERT INTO orders (user_id, status)
    VALUES (10, 'draft')
    RETURNING id
)
INSERT INTO order_items (order_id, product_id, quantity)
SELECT new_order.id, item.product_id, item.quantity
FROM new_order
CROSS JOIN (VALUES (101, 2), (102, 1)) AS item(product_id, quantity);

COMMIT;
```

On failure, the application issues `ROLLBACK`: an order without its items is not retained. Both inserts already form one atomic statement here; an explicit `BEGIN` is needed when other related queries join the operation.

## 3. Errors and SAVEPOINT

After an SQL error inside an explicit transaction, ordinary queries are rejected with `25P02` (`in_failed_sql_transaction`). Issue `ROLLBACK` or roll back to a savepoint created **before the error**.

```sql
BEGIN;
UPDATE orders SET status = 'confirmed' WHERE id = 42;

SAVEPOINT optional_step;
UPDATE orders SET note = 'temporary note' WHERE id = 42;
ROLLBACK TO SAVEPOINT optional_step; -- undo only the note change
RELEASE SAVEPOINT optional_step;

COMMIT; -- the status change is retained
```

This example assumes an `orders.note` column. If the optional step fails with an SQL error, the application can also issue `ROLLBACK TO SAVEPOINT optional_step` and continue when the operation's semantics permit it.

A `SAVEPOINT` provides partial rollback, **not an independent nested transaction**. `RELEASE` does not commit changes: an outer `ROLLBACK` undoes them too. Handle serialization failures and deadlocks using the [whole-transaction retry rules](postgresql-locks-cheatsheet.md#10-deadlocks-waits-and-retry).

## 4. Application Usage

1. Acquire a pooled connection and use it for every query from `BEGIN` through `COMMIT`/`ROLLBACK`.
2. If the ORM provides a transaction callback, execute queries through its `tx`, including calls from nested functions.
3. Await every query and `COMMIT` before reporting success.
4. On error, roll back the transaction and propagate the error to the caller. Release the connection in `finally`; discard a broken connection according to the driver's rules.

**Zero affected rows is not an SQL error.** Check `RETURNING`/row counts; if a required operation did not happen, the application must abort the transaction itself.

## 5. External Effects and Rollback Limits

- An HTTP request, email, or published message does not roll back with database changes. A successful external call does not mean a subsequent `COMMIT` will succeed.
- Sending immediately after `COMMIT` still leaves a failure window: data has been saved but the message has not yet been sent.
- If delivery is required, use an **outbox**: save an event in a table with the data, then let a separate worker deliver it with retries. The recipient must handle duplicates idempotently.
- Sequence values (`serial`/`identity`) are not reclaimed after rollback: gaps in IDs are normal.

## 6. Common Mistakes

| Mistake | Prevention |
|---|---|
| Missing `COMMIT`/`ROLLBACK` | Explicit boundaries and guaranteed transaction completion |
| Catching and hiding an error | Propagate it; continue partially only deliberately through a savepoint |
| Querying through the general pool/ORM instead of `tx` | Pass the transactional connection to every participating function |
| Waiting for HTTP or user actions inside a transaction | Keep the transaction short; coordinate external effects separately |

See also: [locks, timeouts, and diagnostics](postgresql-locks-cheatsheet.md#11-diagnosing-locks), [indexes and CREATE INDEX CONCURRENTLY restrictions](postgresql-indexes-cheatsheet.md#10-design-rules-and-common-mistakes).

### Official Documentation

- [Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)
- [COMMIT](https://www.postgresql.org/docs/current/sql-commit.html)
- [ROLLBACK TO SAVEPOINT](https://www.postgresql.org/docs/current/sql-rollback-to.html)
- [Sequence Functions: rollback limitations](https://www.postgresql.org/docs/current/functions-sequence.html)
