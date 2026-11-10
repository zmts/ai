# Isolation, locking and atomicity

- A transaction is an atomic database boundary, not a guarantee that concurrent reads/writes serialize as intended. PostgreSQL READ COMMITTED normally uses a new statement snapshot; REPEATABLE READ and SERIALIZABLE have different behavior. Check actual isolation.
- Prefer an atomic conditional UPDATE/INSERT with constraints over a read-check-write race. Inspect affected rows/RETURNING; zero rows can mean a stale condition, not success. An UPDATE can wait and recheck under READ COMMITTED.
- SELECT FOR UPDATE locks selected rows, not every possible future row satisfying a predicate. Missing-row invariants often require constraints or stronger coordination.
- Acquire multiple locks in consistent order and keep transactions short. Avoid external IO while holding locks, but moving a side effect outside the transaction alone does not make payment/order processing atomic; preserve idempotency and durable coordination.
- SQLSTATE 40001 (serialization failure) or 40P01 (deadlock) can require retrying the entire transaction, including decisions, within a bounded policy. Do not blindly replay externally visible side effects. Not every database error is transient.
- SKIP LOCKED can serve work-queue claiming, but returns an intentionally inconsistent view and can starve rows. It is not a generic faster SELECT.
- statement_timeout bounds statements, lock_timeout bounds lock waits, and idle_in_transaction_session_timeout bounds idle transactions; none is universally a whole-transaction deadline. Verify version-dependent transaction-level controls before proposing them.
- SET LOCAL belongs inside a transaction. After an error, rollback the transaction or relevant savepoint before normal queries can continue. Application client ownership stays with nodejs-pro.

Sources: [Isolation](https://www.postgresql.org/docs/current/transaction-iso.html), [explicit locks](https://www.postgresql.org/docs/current/explicit-locking.html), [timeouts](https://www.postgresql.org/docs/current/runtime-config-client.html).
