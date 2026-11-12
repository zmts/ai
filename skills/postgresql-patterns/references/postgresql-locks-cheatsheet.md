# PostgreSQL Locks — Cheatsheet

See also: [transactions — boundaries, errors, and savepoints](postgresql-transactions-cheatsheet.md).

## Contents

1. [What a lock is and why it is needed](#1-what-a-lock-is-and-why-it-is-needed)
2. [Optimistic vs pessimistic locking](#2-optimistic-vs-pessimistic-locking)
3. [MVCC and isolation levels](#3-mvcc-and-isolation-levels)
4. [Classifying locks](#4-classifying-locks)
5. [Row-level locks: FOR UPDATE / SHARE](#5-row-level-locks-for-update--share)
6. [NOWAIT and SKIP LOCKED](#6-nowait-and-skip-locked)
7. [Table-level locks](#7-table-level-locks)
8. [Advisory locks](#8-advisory-locks)
9. [Practical cases and choosing an approach](#9-practical-cases-and-choosing-an-approach)
10. [Deadlocks, waits, and retry](#10-deadlocks-waits-and-retry)
11. [Diagnosing locks](#11-diagnosing-locks)

---

## 1. What a Lock Is and Why It Is Needed

A **lock** coordinates concurrent transactions so they do not perform incompatible operations on the same data.

**Example:** two HTTP requests try to debit the same account simultaneously. Without correct concurrency logic, the result can be an incorrect balance or a lost update.

- **Lock ≠ transaction:** a transaction groups operations and defines commit boundaries; a lock protects conflicting actions.
- **Lock ≠ index:** an index speeds up lookup but does not itself solve concurrent modification.
- PostgreSQL automatically acquires the required locks for `INSERT`, `UPDATE`, `DELETE`, and DDL. Explicit locking is not always necessary.

---

## 2. Optimistic vs Pessimistic Locking

These are **two strategies**, not SQL lock-mode names.

| Criterion | Optimistic | Pessimistic |
|---|---|---|
| Idea | Conflicts are rare: check when writing | Conflicts are likely: lock beforehand |
| Read | No lock retained between the read and write | Usually `SELECT ... FOR UPDATE` |
| Conflict detection | Through `version` / an `UPDATE` condition | When attempting to acquire an incompatible lock |
| Behavior on conflict | `0 rows` → retry / 409 Conflict | Wait / `NOWAIT` / timeout |
| Suitable for | Profiles, settings, forms | Balances, reservations, queue processing |
| Risks | Conflicts under frequent writes | Waits, deadlocks, long transactions |

### Optimistic: Version Check

```sql
-- 1. The client read a record with version = 5
SELECT id, balance, version
FROM accounts
WHERE id = 1;

-- 2. Change it only if the version is unchanged
UPDATE accounts
SET balance = 900,
    version = version + 1
WHERE id = 1
  AND version = 5
RETURNING id, balance, version;
```

- **A row returned** → the change succeeded.
- **0 rows** → the version changed **or the record is absent**; handle the conflict, reread the data, or retry according to business rules.
- Even an optimistic `UPDATE` takes ordinary internal PostgreSQL locks while executing; “optimistic” means **no pre-acquired lock is retained between reading and writing**.

### Pessimistic: Lock Before Read-Modify-Write

```sql
BEGIN;

SELECT balance
FROM accounts
WHERE id = 1
FOR UPDATE;

-- Check business conditions and calculate the new value
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

COMMIT;
```

- A conflicting transaction waits for the row to be released.
- The lock is held **until the transaction ends** (`COMMIT` / `ROLLBACK`).
- An ordinary `SELECT` without a locking clause can usually continue reading the visible row version through MVCC.

---

## 3. MVCC and Isolation Levels

**MVCC (Multi-Version Concurrency Control)** means PostgreSQL keeps row versions and uses snapshots. As a result, an ordinary `SELECT` need not wait for a concurrent `UPDATE`, and vice versa.

| Isolation level | PostgreSQL behavior | Purpose |
|---|---|---|
| `READ UNCOMMITTED` | Implemented as `READ COMMITTED` | No separate weaker mode |
| **`READ COMMITTED`** | Snapshot per statement; default | Typical CRUD |
| **`REPEATABLE READ`** | Transaction snapshot; PostgreSQL prevents phantom reads | Consistent reads |
| **`SERIALIZABLE`** | SSI checks whether execution can be serialized | Complex invariants across multiple rows |

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- related reads and writes
COMMIT;
```

**Important:** `SERIALIZABLE` can raise `40001` (`serialization_failure`). The application must be able to **retry the whole transaction**, not only the last SQL statement. PostgreSQL also uses `SIReadLock` (predicate locking) for SSI; unlike `FOR UPDATE`, it does not block ordinary writes.

---

## 4. Classifying Locks

| Dimension | Options | Meaning |
|---|---|---|
| Strategy | Optimistic / Pessimistic | Check for conflict on write or lock beforehand |
| Compatibility | Shared / Exclusive | Whether other transactions can acquire conflicting locks |
| Object | Row / Table / Advisory | A row, a table, or an application-defined logical key |
| Lifetime | Transaction / Session | Until transaction or session end |
| Behavior when busy | Wait / `NOWAIT` / `SKIP LOCKED` | Wait, fail, or skip busy rows |
| Additional mechanism | Predicate / `SIReadLock` | Track serialization conflicts under SSI |

> PostgreSQL has no universal modes literally named `S`/`X` for every situation: row-level and table-level locks have their own **specific modes and compatibility matrices**.

---

## 5. Row-Level Locks: FOR UPDATE / SHARE

| Mode | Purpose | Important characteristic |
|---|---|---|
| **`FOR UPDATE`** | Modify / delete the selected row | Strongest of the four row-lock modes |
| **`FOR NO KEY UPDATE`** | Update without changing key fields | Weaker than `FOR UPDATE`; allows `FOR KEY SHARE` |
| **`FOR SHARE`** | Stabilize rows for subsequent logic | Multiple `FOR SHARE` locks are compatible |
| **`FOR KEY SHARE`** | Protect the key from deletion/change | Does not block `FOR NO KEY UPDATE` |

```sql
BEGIN;
SELECT * FROM orders WHERE id = 42 FOR UPDATE;
-- order logic and modification
UPDATE orders SET status = 'paid' WHERE id = 42;
COMMIT;
```

**Important details:**

- `FOR UPDATE` does not mean “prevent every read”: an ordinary `SELECT` can read an older visible version.
- `UPDATE` automatically acquires a row lock: usually `FOR NO KEY UPDATE`, or the stronger `FOR UPDATE` when changing certain unique key columns.
- `SELECT ... FOR UPDATE` over many rows increases the number of conflicting objects. Narrow `WHERE` and keep the transaction short.
- **`FOR UPDATE` does not lock a missing row.** If a check for “no record yet” returns no rows, a concurrent `INSERT` is still possible. Use `UNIQUE` for uniqueness, or suitable coordination or `SERIALIZABLE` with retry for more complex invariants.

### Simplified Row-Lock Compatibility

`✓` means compatible; `✕` means conflicting, for different transactions on the same row.

| Requested ↓ / Already held → | KEY SHARE | SHARE | NO KEY UPDATE | UPDATE |
|---|:---:|:---:|:---:|:---:|
| **KEY SHARE** | ✓ | ✓ | ✓ | ✕ |
| **SHARE** | ✓ | ✓ | ✕ | ✕ |
| **NO KEY UPDATE** | ✓ | ✕ | ✕ | ✕ |
| **UPDATE** | ✕ | ✕ | ✕ | ✕ |

---

## 6. NOWAIT and SKIP LOCKED

| Variant | Behavior | When to use |
|---|---|---|
| Ordinary `FOR UPDATE` | Waits for lock release | A particular row must be processed |
| `FOR UPDATE NOWAIT` | Immediately raises `55P03` if the row is busy | Fail-fast APIs / resource acquisition attempts |
| `FOR UPDATE SKIP LOCKED` | Omits rows locked by others | A multi-worker job queue |

```sql
-- Do not wait for a busy row
SELECT * FROM jobs
WHERE id = 5
FOR UPDATE NOWAIT;

-- Different workers claim different jobs
SELECT * FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 10
FOR UPDATE SKIP LOCKED;
```

**Details:** `SKIP LOCKED` provides an **incomplete/inconsistent result**, so it usually fits queues rather than ordinary reporting. It can also starve repeatedly locked rows; it is not a generic faster SELECT. `NOWAIT` and `SKIP LOCKED` apply to **row-level** locks; the required table-level `ROW SHARE` lock is still acquired normally.

### Atomic Job Claiming

```sql
WITH picked AS (
  SELECT id
  FROM jobs
  WHERE status = 'pending'
  ORDER BY id
  LIMIT 10
  FOR UPDATE SKIP LOCKED
)
UPDATE jobs AS j
SET status = 'processing'
FROM picked
WHERE j.id = picked.id
RETURNING j.*;
```

Run this in a short transaction, then commit. For long processing, store a lease/timeout and a mechanism to reissue stuck jobs instead of keeping the transaction open throughout the worker's execution.

---

## 7. Table-Level Locks

**Table locks** are acquired explicitly (`LOCK TABLE`) and automatically by SQL operations. The name `ROW EXCLUSIVE` **does not mean** a single-row lock: it is a **table** lock mode.

| Mode | Typical source | What to watch |
|---|---|---|
| `ACCESS SHARE` | Ordinary `SELECT` | Conflicts with `ACCESS EXCLUSIVE` |
| `ROW SHARE` | `SELECT ... FOR UPDATE/SHARE` | Acquired in addition to row locks |
| `ROW EXCLUSIVE` | `INSERT`, `UPDATE`, `DELETE` | Does not block ordinary `SELECT` |
| `SHARE UPDATE EXCLUSIVE` | `VACUUM` without FULL, some DDL | Restricts some competing maintenance operations |
| `SHARE` | Ordinary `CREATE INDEX` | Conflicts with modifying transactions |
| `SHARE ROW EXCLUSIVE` | Some DDL operations | A strong self-conflicting lock |
| `EXCLUSIVE` | Some explicit operations | Allows ordinary `SELECT` but blocks many other modes |
| `ACCESS EXCLUSIVE` | `TRUNCATE`, `DROP TABLE`, some `ALTER TABLE` | Blocks even ordinary `SELECT` |

```sql
BEGIN;
LOCK TABLE orders IN SHARE MODE;
-- actions requiring this mode
COMMIT;
```

**Ordinary `CREATE INDEX` can block writes;** `CREATE INDEX CONCURRENTLY` usually allows concurrent `INSERT/UPDATE/DELETE`, but has restrictions and runs **outside a transaction block**.

---

## 8. Advisory Locks

An **advisory lock** is a logical lock on a numeric key. PostgreSQL does not automatically associate it with a particular row: **all participants must voluntarily follow the same key protocol**.

| API | Lifetime | Usage |
|---|---|---|
| `pg_advisory_xact_lock(key)` | Until `COMMIT`/`ROLLBACK` | Protect an operation inside a transaction |
| `pg_try_advisory_xact_lock(key)` | Until transaction end | Try without waiting |
| `pg_advisory_lock(key)` | Until release / **session** end | A long-lived logical mutex |
| `pg_try_advisory_lock(key)` | Until release / session end | A non-blocking attempt |
| `pg_advisory_unlock(key)` | Explicit release | Session-scoped locks only |

```sql
BEGIN;
SELECT pg_advisory_xact_lock(1001);
-- critical section
COMMIT; -- releases automatically
```

```sql
-- Session-scoped: use care with connection pools
SELECT pg_advisory_lock(1001);
-- operations
SELECT pg_advisory_unlock(1001);
```

- **Shared** advisory-lock variants exist; a key can be one `bigint` or a pair of `int` values.
- Transaction-scoped variants are usually safer in backend code with connection pooling.
- A session-scoped lock belongs to the **physical connection**, not the HTTP request. Returning the connection to the pool can accidentally leave the lock held.
- Advisory locks do not guarantee “one process forever”: connection failure releases the lock. Reliable background jobs also benefit from leases, idempotency, and state checks.

---

## 9. Practical Cases and Choosing an Approach

Prefer an atomic conditional `UPDATE`/`INSERT` backed by constraints over a read-check-write race when it expresses the required invariant. Inspect affected rows or `RETURNING`; zero rows can mean a stale condition rather than success.

| Task | Recommended approach | Reason |
|---|---|---|
| Two clients editing a profile | Optimistic `version` | Do not keep a transaction open between reading and saving the form |
| A simple balance debit | **One atomic `UPDATE`** | Often simpler and safer than a preliminary `SELECT` |
| Complex read-modify-write with business conditions | `SELECT ... FOR UPDATE` | Stabilize specific rows while checking |
| An SQL job queue with several workers | `FOR UPDATE SKIP LOCKED` | Workers do not wait for the same jobs |
| One logical operation per resource | Transaction-scoped advisory lock | The key need not correspond to one row |
| An invariant across several rows | `SERIALIZABLE` + retry, or carefully designed explicit coordination | Protect against write skew and other anomalies |
| Analytical reads only | MVCC / a suitable isolation level | `FOR UPDATE` is usually unnecessary |

### Example: Atomic Debit, Usually Better Than Two Queries

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1
  AND balance >= 100
RETURNING id, balance;
```

- **A row returned** → the debit succeeded.
- **0 rows** → insufficient funds **or the account was not found**.
- PostgreSQL acquires the required `UPDATE` locks itself. Under `READ COMMITTED`, an update can wait and recheck its condition against a concurrently updated row; a prior read does not guarantee that the later condition still holds.
- To **transfer** money between two accounts, combine the debit and credit **in one transaction**; access accounts in a consistent order to reduce deadlock risk.

### Example: Optimistic Locking in an API

```sql
UPDATE documents
SET content = $1,
    version = version + 1
WHERE id = $2
  AND version = $3
RETURNING id, version;
```

Check `rowCount`. If it is `0`, return a conflict, often HTTP `409`, and suggest rereading the record rather than silently overwriting someone else's changes.

### Example: Locking Several Accounts

```sql
BEGIN;

-- Lock in a stable order by id
SELECT id, balance
FROM accounts
WHERE id IN (10, 20)
ORDER BY id
FOR UPDATE;

-- Check the balance and update both accounts
COMMIT;
```

**Do not keep** a transaction open during network calls to a payment API: delays create a queue of waiters and increase deadlock/timeout risk. Moving an external effect outside a transaction alone does not make the overall operation atomic; preserve idempotency and durable coordination, as described in the transactions cheatsheet.

---

## 10. Deadlocks, Waits, and Retry

A **deadlock** is a cycle of waits:

```text
TX1: holds lock(A) → waits for lock(B)
TX2: holds lock(B) → waits for lock(A)
```

PostgreSQL detects the deadlock and aborts one transaction with SQLSTATE **`40P01`**.

**Prevention:**

1. Always acquire resources in the same order (`id ASC`).
2. Keep transactions short, without external HTTP requests inside.
3. Update/lock only required rows; index `WHERE` appropriately.
4. Bound waits with `lock_timeout` when needed.
5. Retry the **whole transaction**, including its decisions, on `40P01` and `40001`, with bounded attempts, time budget, and backoff. Do not blindly replay externally visible side effects or classify every database failure as transient.

```sql
-- Example: bound lock waiting for the transaction
BEGIN;
SET LOCAL lock_timeout = '2s';
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
COMMIT;
```

| SQLSTATE | Meaning | Action |
|---|---|---|
| `40P01` | `deadlock_detected` | Retry the whole transaction |
| `40001` | `serialization_failure` | Retry the whole transaction |
| `55P03` | `lock_not_available` (`NOWAIT`/`lock_timeout`) | Return busy; retry later according to API rules |

---

## 11. Diagnosing Locks

### Which Locks Are Held or Awaited

```sql
SELECT
    pid,
    locktype,
    relation::regclass AS relation,
    mode,
    granted
FROM pg_locks
WHERE relation = 'orders'::regclass;
```

`granted = false` means the lock is **being awaited**.

Filtering by `relation` does not show every row-related wait: these often appear as waits on `transactionid`, without a table reference. Use the following `pg_blocking_pids()` query to find the blocking process.

### Who Is Blocking a Process

```sql
SELECT
    pid,
    state,
    wait_event_type,
    wait_event,
    pg_blocking_pids(pid) AS blocked_by,
    query
FROM pg_stat_activity
WHERE wait_event_type = 'Lock';
```

### Quick Settings

```sql
SHOW lock_timeout;
SHOW deadlock_timeout;
SHOW statement_timeout;
```

- `lock_timeout`: how long to wait **to acquire a lock**.
- `statement_timeout`: a limit on the **whole statement**.
- `deadlock_timeout`: delay before checking for a deadlock, usually an operational database setting.

**`idle in transaction` can also retain locks.** Check the blocker's PID in `pg_stat_activity`, its `state`, and `xact_start`. `statement_timeout` does not bound idle time between queries; `idle_in_transaction_session_timeout` exists for that purpose and terminates the session. After an SQL error in an explicit transaction, issue `ROLLBACK` before another attempt.

These timeout scopes are different: statement, lock-wait, and idle-transaction limits are not a universal whole-transaction deadline. Check the installed PostgreSQL major for transaction-level controls before recommending them. `SET LOCAL` belongs inside a transaction. After an error, roll back the whole transaction or the relevant savepoint before issuing normal queries. Application connection/client ownership remains with nodejs-pro.

---

### Official Documentation

- [Explicit Locking — PostgreSQL](https://www.postgresql.org/docs/current/explicit-locking.html)
- [Transaction Isolation — PostgreSQL](https://www.postgresql.org/docs/current/transaction-iso.html)
- [MVCC Introduction — PostgreSQL](https://www.postgresql.org/docs/current/mvcc-intro.html)
- [Advisory Lock Functions — PostgreSQL](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADVISORY-LOCKS)
- [Monitoring Database Activity — PostgreSQL](https://www.postgresql.org/docs/current/monitoring-stats.html)
- [pg_locks — PostgreSQL](https://www.postgresql.org/docs/current/view-pg-locks.html)
- [Client Connection Defaults — PostgreSQL](https://www.postgresql.org/docs/current/runtime-config-client.html)

> Note: SQL examples are illustrative. Check the chosen mode, waiting behavior, and behavior under load using transactions on your PostgreSQL version.
