# Sources and local adaptation

Reviewed 2026-10-20. Local version 1.0.0 identifies this curated skill, not an upstream release.

- [Supabase Postgres Best Practices](https://github.com/supabase/agent-skills/tree/32912161e2732c3e5001c6811a76c1f8308ed0da/skills/supabase-postgres-best-practices), pinned commit `32912161e2732c3e5001c6811a76c1f8308ed0da`, declared skill 1.1.1. Inspected the entrypoint and references for missing/partial indexes, short transactions and data types.
- [Jeffallan SQL Pro](https://github.com/Jeffallan/claude-skills/tree/958e60fb998e008507c978b7943211f337def6a6/skills/sql-pro), pinned commit `958e60fb998e008507c978b7943211f337def6a6`. Compared the entrypoint's query/plan workflow and constraints.
- Local text and SQL examples are independently written, with PostgreSQL primary documentation linked by topic. Upstream reference/code files are not vendored. Both inspected upstream skill frontmatters declare MIT.

## Corrections and boundaries

- No blanket prohibition of sequential scans, mandatory indexes on all filters/foreign keys, universal 100ms target or invented speedups.
- No compulsory bigint IDs, timestamptz for intentional wall times, numeric(10,2) for every monetary value, or new schema/ORM tools.
- Explicitly distinguish EXPLAIN estimates from execution, statement versus transaction timeouts, constraints versus pre-checks, result multiplicity and ordered semantics.
- Short transactions do not alone make external side effects atomic. Index CONCURRENTLY still has lock/failure/transaction-block constraints.
- Access routing follows the user's selected tool and repository instructions. Skill installation does not authorize a live connection or production database changes.
- Node client/pool ownership remains in nodejs-pro; SQL, isolation and schema semantics live here. Non-PostgreSQL dialect coverage is outside this revision.

Use documentation for the target PostgreSQL major; current links can move to a newer major. Compare updates to these pinned sources and preserve local corrections and authorization. Newer dates do not prove better behavior.

## Validation for this revision

Structural validation passed for frontmatter, UI metadata, local references and fenced examples. Both documented SQL examples executed with expected rows in a disposable PostgreSQL 17.11 container. Additional checks covered NULL/NOT IN versus NOT EXISTS, exact numeric arithmetic, a Europe/Kyiv DST day boundary, rollback, conditional UPDATE, primary-key/CHECK/NOT NULL rejection and EXPLAIN ANALYZE JSON parsing.

The isolated container had no project data mounts or exposed ports and was removed after testing. No project database connection was made. Its configured image is not evidence of the live server major. Multi-version behavior, concurrent-session races, RLS policy enforcement and production DDL/lock impact were not integration-tested.

## Reference expansion — 2026-11-12

Local version 1.1.0 adds English versions of the four PostgreSQL cheatsheets maintained in the repository's `info/` directory: data modeling/JOINs, indexes, transactions and locks. The Russian originals remain unchanged. SQL statements are retained; comments, explanations and navigation are translated.

The separate short plan/index and concurrency references are consolidated into the index and lock cheatsheets. Their unique safeguards remain: execution boundaries and plan interpretation, prepared-plan limitations, evidence-based performance claims, bounded retry, lock starvation and distinct timeout scopes. SQL correctness, types/time, schema/migrations and access references remain separate because they cover additional concerns.

Version-sensitive index and lock details were checked against PostgreSQL 18 primary documentation. Source links using `current` still need to be matched to the actual server major. Index naming is an illustrative project convention, not a global requirement. SQL examples are alternatives, not a single sequential migration.

Validation for this expansion is recorded separately from the earlier container tests above: translation coverage, unchanged Russian originals and SQL statement bodies, local links/anchors, skill structure and SQL parsing. No live database, DDL, maintenance or concurrent-session workload is required for these documentation checks.
