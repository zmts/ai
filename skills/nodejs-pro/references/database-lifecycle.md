# Database connections and transactions

- Reuse the configured pool; avoid a pool per request. Budget total connections across API replicas and workers, not only one process.
- A transaction must use one checked-out connection (or the ORM's transaction object) for BEGIN, statements and COMMIT/ROLLBACK. pool.query does not guarantee the same connection across calls.
- Always release checked-out clients. On rollback failure, discard the damaged connection and preserve both failures. Observe pool background errors using the established logger/recovery policy.
- Use the ORM transaction API where already established; do not wrap it in a competing transaction system. Propagate its transaction handle to participating operations.
- Do not hold a transaction while making unrelated slow HTTP calls. Concurrent promises on one connection do not create parallel database transactions.
- Parameterize values. Dynamic identifiers require a trusted allowlist or identifier quoting; value placeholders do not escape table/column names. A pre-insert uniqueness lookup does not replace a database constraint.
- Connection release is per operation; pool.end belongs to the process/resource owner after dependent work drains.
- This reference covers lifecycle, not a SQL optimization or migration workflow. Use the user's designated database access method; do not connect to a real database just to test this skill.
- For query correctness, plans/indexes, PostgreSQL isolation/locking or schema changes, use postgresql-patterns if available. Do not load it solely to release an application client.

Illustrative node-postgres helper. Reuse an existing transaction abstraction instead when present. The callback must await all its work before returning and must not release its borrowed client.

```javascript
async function withTransaction(pool, work) {
  const client = await pool.connect();
  let discard;
  try {
    await client.query('BEGIN');
    const result = await work(client);
    await client.query('COMMIT');
    return result;
  } catch (originalError) {
    try {
      await client.query('ROLLBACK');
    } catch (rollbackError) {
      discard = rollbackError instanceof Error ? rollbackError : new Error('Rollback failed');
      throw new AggregateError([originalError, rollbackError], 'Transaction and rollback failed');
    }
    throw originalError;
  } finally {
    client.release(discard);
  }
}
```

Sources: [node-postgres transactions](https://node-postgres.com/features/transactions), [pooling](https://node-postgres.com/features/pooling).
