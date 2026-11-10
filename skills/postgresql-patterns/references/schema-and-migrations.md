# Constraints, JSONB, RLS and migrations

- Preserve domain invariants with appropriate NOT NULL, CHECK, UNIQUE, foreign keys and transactional logic. CHECK expressions evaluating to NULL pass; use NOT NULL separately when required. UNIQUE NULL behavior and NULLS NOT DISTINCT availability are version-sensitive.
- Prefer explicit columns for stable relational fields. JSONB is useful for genuinely variable structure; SQL NULL, JSON null and a missing key differ. -> returns JSON, ->> text. Validate casts and index the actual access operators; not every GIN operator class supports every predicate.
- RLS is authorization, not a replacement for grants/role design. Test policies as the actual application role; owners/superusers/BYPASSRLS can bypass policies under relevant conditions. SECURITY DEFINER needs controlled privileges/search_path; do not add or ban it categorically.
- Inspect migration tooling, table size, lock acquisition, dependencies, replication and compatibility with old/new application versions before DDL. Small syntax does not mean low operational impact.
- CREATE INDEX CONCURRENTLY avoids blocking ordinary writes in the same way as a regular build, but still acquires locks, can wait/fail, and leaves invalid indexes on some failures. It cannot run inside a transaction block. Coordinate with migration runners and verify validity; do not blindly retry IF NOT EXISTS on an invalid index.
- ALTER TABLE may take strong locks; not all changes rewrite tables, and defaults/type changes differ by version. Use staged expand/backfill/validate/contract only when scale and compatibility justify it.
- Backfills need bounded batches, stable progress, idempotency and a retry policy. Do not silently transform a diagnostic task into an unbounded UPDATE or schema rewrite.
- Rollback is not a backup, and down migrations can lose data. Explain concrete operational effects before executing schema/data changes under the user's authorization.

Sources: [Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html), [JSON](https://www.postgresql.org/docs/current/datatype-json.html), [RLS](https://www.postgresql.org/docs/current/ddl-rowsecurity.html), [CREATE INDEX](https://www.postgresql.org/docs/current/sql-createindex.html), [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html).
