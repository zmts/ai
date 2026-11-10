---
name: postgresql-patterns
description: Write or review substantive PostgreSQL SQL, diagnose query plans and indexes, or design database constraints, JSONB usage, migrations, and transaction isolation/locking. Use for SQL/database semantics; routine Node connection cleanup, generic API changes, and non-PostgreSQL dialects have other workflows.
metadata:
  version: "1.0.0"
---

# PostgreSQL Patterns

Preserve query correctness first, then improve measured database behavior within the authorized scope.

## Workflow

1. Establish the target, PostgreSQL version, schema/constraints, relevant query and expected result. Inspect local config/migrations first; do not infer a live server version from an image tag. Use the project's designated access method.
2. Trace row cardinality, NULL semantics, types, ordering, time boundaries and transaction context before rewriting SQL.
3. Read only the reference matching the task below. Start performance analysis with evidence; plan estimates are not measured execution times.
4. Keep diagnosis separate from schema/data changes. A request for advice or analysis does not authorize migrations, index creation, maintenance or benchmark writes.
5. Validate result equivalence and the affected failure case with representative task-owned data; compare performance only under comparable conditions. Report actual observations and limitations.

## Decision rules

- Parameterize values; allowlist/quote identifiers through the existing driver. Do not embed untrusted SQL in shell command text.
- EXPLAIN ANALYZE executes the statement, including writes, locks, triggers and function side effects. Read-only SQL can still invoke side-effecting functions. A rollback does not undo every external/sequence effect.
- Sequential scans and sorts are not inherently defects. No universal latency target, index-per-filter rule, forced plan or fabricated speedup.
- Preserve duplicates, missing groups, NULL handling, precision and deterministic ordering when optimizing. Verify pagination ties and window frames.
- Consider write/storage/locking costs and installed-version support before recommending indexes or migrations.
- Do not require Supabase hosting, a new ORM/connector or a real database connection for conceptual answers.
- Use nodejs-pro for application connection/client ownership if available; this skill owns SQL semantics, plans and PostgreSQL concurrency, not a second pool implementation.

## References by task

| Task | Reference |
|---|---|
| JOINs, CTEs, aggregates, windows, NULL and pagination | [SQL correctness](references/sql-correctness.md) |
| Numeric precision, timestamps and date boundaries | [Types and time](references/types-and-time.md) |
| EXPLAIN, statistics, indexes and performance claims | [Plans and indexes](references/plans-and-indexes.md) |
| Isolation, locks, retries and atomic updates | [Concurrency](references/concurrency.md) |
| Constraints, JSONB, RLS and schema migrations | [Schema and migrations](references/schema-and-migrations.md) |
| Database access and execution boundaries | [Access](references/access.md) |
| Pinned source revisions and local adaptations | [Sources](references/sources.md) |

Return the SQL/recommendation, its semantic effect, validation evidence and material execution limits. Keep scope and output proportional to the task.
