# PostgreSQL Indexes

See also: [transactions — boundaries, errors, and rollback](postgresql-transactions-cheatsheet.md).

See also: [normalization, denormalization, and JOINs](postgresql-data-modeling-joins-cheatsheet.md).

## Contents

1. [What an index is](#1-what-an-index-is)
2. [B-tree: how it works and why it is fast](#2-b-tree-how-it-works-and-why-it-is-fast)
3. [PostgreSQL index types](#3-postgresql-index-types)
4. [Index variants by purpose](#4-index-variants-by-purpose)
5. [Example table and multicolumn index](#5-example-table-and-multicolumn-index)
6. [Which SELECTs use a multicolumn index](#6-which-selects-use-a-multicolumn-index)
7. [How PostgreSQL combines multiple indexes](#7-how-postgresql-combines-multiple-indexes)
8. [Practical examples of other indexes](#8-practical-examples-of-other-indexes)
9. [Checking an index with EXPLAIN](#9-checking-an-index-with-explain)
10. [Design rules and common mistakes](#10-design-rules-and-common-mistakes)
11. [Big O: algorithmic complexity](#11-big-o-algorithmic-complexity)

---

## 1. What an Index Is

An **index** is an additional data structure that can find table rows faster without scanning the whole table (`Seq Scan`).

**Analogy:** a book index: first find the topic in the index, then the required page.

```sql
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'a@example.com';
```

An index stores keys and references to table rows. It **does not guarantee** that PostgreSQL will choose it: the planner estimates read cost, statistics, the number of matching rows, and other factors.

**Index cost:** additional disk space and overhead on `INSERT`, `UPDATE`, and `DELETE`.

## 2. B-tree: How It Works and Why It Is Fast

A **B-tree (Balanced Tree)** is a balanced tree with sorted keys. PostgreSQL uses a B-tree organization close to a **B+ tree**: index data is organized into pages, and lookup proceeds from the root to leaf pages.

```text
                   [50]
                 /      \
           [10 | 30]  [70 | 90]
           /    ...     ...   \
        [5]   [25]    [60]   [95]
```

*A simplified illustration of branching, not PostgreSQL's exact page format.*

- **Foundation:** sorted keys, multiple branches per node, and balanced depth.
- **Lookup:** at each level, choose the relevant branch and discard other ranges.
- **Complexity:** approximately `O(log N)` for lookup, insertion, and deletion.
- **Why balance matters:** the tree does not degenerate into a long chain.
- **Suitable for:** `=`, `<`, `>`, `<=`, `>=`, `BETWEEN`, often `IN`, `ORDER BY`, and some prefix searches.

```sql
CREATE INDEX idx_users_age ON users(age);
SELECT * FROM users WHERE age = 25;
SELECT * FROM users WHERE age BETWEEN 20 AND 30;
SELECT * FROM users ORDER BY age LIMIT 10;
```

## 3. PostgreSQL Index Types

| Type | Useful for | Typical operations / characteristics |
|---|---|---|
| **B-tree** (default) | Ordinary fields, sorting, ranges | `=`, `<`, `>`, `BETWEEN`, `IN`, `ORDER BY` |
| **Hash** | Exact equality only | `=`; B-tree is usually more versatile |
| **GIN** | `jsonb`, arrays, full-text search | `@>`, `?`, `&&`, `@@`, depending on data type and operator class |
| **GiST** | Geometry, ranges, nearest-neighbor search | `&&`, `<@`, `<->`, when supported by the operator class |
| **SP-GiST** | Space-partitioning structures | For example, point lookup or prefix structures; depends on the operator class |
| **BRIN** | Huge tables with correlation between values and physical order | A compact index over page ranges, for example on `created_at` |

> The **operator class** determines specific operators. Not every listed operator applies to every data type.

## 4. Index Variants by Purpose

These are **not separate access methods**, but ways of defining an index, usually B-tree.

| Variant | Purpose |
|---|---|
| **Unique** | Enforce key uniqueness |
| **Composite / Multicolumn** | Several columns in one index |
| **Partial** | Index only rows satisfying a `WHERE` predicate |
| **Expression** | Index an expression, such as `lower(email)` |
| **Covering (`INCLUDE`)** | Add non-key columns, potentially enabling an `Index Only Scan` |

```sql
-- Uniqueness
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Multicolumn
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial
CREATE INDEX idx_orders_active ON orders(user_id)
WHERE status = 'active';

-- Expression
CREATE INDEX idx_users_email_lower ON users(lower(email));

-- Covering
CREATE INDEX idx_orders_cover ON orders(user_id) INCLUDE (status, amount);
```

`INCLUDE` does not make the included columns part of the search/sort key. An `Index Only Scan` can avoid heap access only when the index covers the required data **and** row visibility can be established through the visibility map.

## 5. Example Table and Multicolumn Index

```sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    user_id INT,
    status VARCHAR(20),
    amount NUMERIC,
    created_at TIMESTAMP
);

CREATE INDEX idx_orders_user_status_date
ON orders(user_id, status, created_at);
```

Key order:

```text
user_id → status → created_at
   1st      2nd        3rd
```

The index is ordered **first by `user_id`, then by `status` within that user, then by `created_at` within that group**.

### Main Rule for a Multicolumn B-tree

The index narrows lookup most effectively when the query constrains a **leftmost prefix** of keys, especially with equality on leading columns. Skipping an intermediate column usually reduces lookup/sort efficiency.

## 6. Which SELECTs Use a Multicolumn Index

Assume the only index is:

```sql
CREATE INDEX idx_orders_user_status_date
ON orders(user_id, status, created_at);
```

| Query condition | Will the index help? | Reason |
|---|---|---|
| `user_id = 10` | ✅ Usually effective | First column |
| `user_id = 10 AND status = 'paid'` | ✅ Usually effective | First two columns |
| `user_id = 10 AND status = 'paid' AND created_at > ...` | ✅ Usually effective | Equalities plus a range |
| `user_id = 10 AND created_at > ...` | ⚠️ Partly | No condition on `status` |
| `status = 'paid'` | ⚠️ Usually limited | First column skipped |
| `created_at > ...` | ⚠️ Usually limited | First two columns skipped |

*⚠️ Does not mean “the index is never used.” The planner may choose a full/partial index scan, skip scan (PostgreSQL 18+, with a suitable data distribution), or `Seq Scan`.*

**A. Lookup by the first column — a good case:**

```sql
SELECT * FROM orders WHERE user_id = 10;
```

**B. Lookup by the first two columns — a good case:**

```sql
SELECT * FROM orders
WHERE user_id = 10 AND status = 'paid';
```

**C. Equalities plus a range on the third column — a good case:**

```sql
SELECT * FROM orders
WHERE user_id = 10
  AND status = 'paid'
  AND created_at > '2026-01-01';
```

**D. Filtering plus sorting — a good case:**

```sql
SELECT * FROM orders
WHERE user_id = 10 AND status = 'paid'
ORDER BY created_at DESC
LIMIT 20;
```

A B-tree can be scanned backward to satisfy `DESC` without an additional sort.

**E. `status` is skipped — partial help:**

```sql
SELECT * FROM orders
WHERE user_id = 10
  AND created_at > '2026-01-01';
```

The index groups rows efficiently by `user_id`, but `created_at` does not form one contiguous range across different `status` values.

**F. Only `status` — usually a weak case:**

```sql
SELECT * FROM orders WHERE status = 'paid';
```

If this query is frequent and selective enough, consider a separate index:

```sql
CREATE INDEX idx_orders_status ON orders(status);
```

**A note on ranges:**

```sql
SELECT * FROM orders
WHERE user_id > 10 AND status = 'paid';
```

Because the first key has a range condition, the second key usually cannot narrow the scan as effectively as with `user_id = 10 AND status = 'paid'`.

## 7. How PostgreSQL Combines Multiple Indexes

Separate and multicolumn indexes can be defined:

```sql
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

| Query | Possible plan |
|---|---|
| `WHERE user_id = 10` | `idx_orders_user` **or** the multicolumn index |
| `WHERE status = 'paid'` | `idx_orders_status` |
| `WHERE user_id = 10 AND status = 'paid'` | The multicolumn index **or** `BitmapAnd` of two separate indexes |
| `WHERE user_id = 10 OR status = 'paid'` | `BitmapOr` over two separate indexes |

**BitmapAnd / BitmapOr:** PostgreSQL can first obtain matching row sets through different indexes, intersect (`AND`) or union (`OR`) them, then read the table rows. Important: bitmap access **generally loses key order**, so `ORDER BY` may need a separate sort.

**Do not create all three indexes automatically:** each consumes space and makes writes more expensive. Choose based on real queries and `EXPLAIN`.

## 8. Practical Examples of Other Indexes

### GIN for JSONB

```sql
CREATE TABLE products (id BIGSERIAL PRIMARY KEY, metadata JSONB);
CREATE INDEX idx_products_metadata ON products USING GIN (metadata);

SELECT * FROM products
WHERE metadata @> '{"color":"red"}';
```

### GIN + pg_trgm for Substring Search

An ordinary B-tree generally does not help with a leading `%` in `LIKE '%term%'`.

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_users_name_trgm
ON users USING GIN (name gin_trgm_ops);

SELECT * FROM users WHERE name ILIKE '%alex%';
```

### BRIN for a Large Event Log

```sql
CREATE INDEX idx_events_created_at
ON events USING BRIN (created_at);

SELECT * FROM events
WHERE created_at >= '2026-01-01'
  AND created_at < '2026-02-01';
```

BRIN is especially useful when new events arrive roughly in time order and are physically close. It is smaller than B-tree but may read extra blocks and recheck conditions afterward.

### Partial Index

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';

SELECT * FROM orders
WHERE user_id = 10 AND status = 'active';
```

A partial index helps when query conditions **imply** its predicate. It does not suit an arbitrary `status` without that condition; with parameterized queries, the planner may also be unable to prove the implication.

### Expression Index

```sql
CREATE INDEX idx_users_lower_email ON users(lower(email));

SELECT * FROM users WHERE lower(email) = 'a@example.com';
```

An ordinary index on `email` is not equivalent to an index on `lower(email)`.

### Covering Index (`INCLUDE`)

```sql
CREATE INDEX idx_orders_user_cover
ON orders(user_id) INCLUDE (status, amount);

SELECT status, amount FROM orders WHERE user_id = 10;
```

This can enable an `Index Only Scan`, with the visibility map determining whether heap access can be avoided.

## 9. Checking an Index with EXPLAIN

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders
WHERE user_id = 10 AND status = 'paid';
```

| Plan element | Meaning |
|---|---|
| `Seq Scan` | Sequential table access |
| `Index Scan` | Access matching rows through the index and table |
| `Index Only Scan` | Data comes from the index, but heap fetches may still be needed |
| `Bitmap Index Scan` | The index builds a bitmap of matching rows |
| `Bitmap Heap Scan` | Table blocks are read using the bitmap |
| `BitmapAnd` / `BitmapOr` | Combine row sets from indexes |
| `Sort` | An additional sort |

Inspect `actual time`, `rows`, `loops`, `Buffers`, `Index Cond`, `Filter`, and `Rows Removed by Filter`. Distinguish an **index search condition** (`Index Cond`) from a **post-retrieval filter** (`Filter`).

> `EXPLAIN ANALYZE` really executes the query. For `UPDATE`/`DELETE`, use test data or a transaction with `ROLLBACK`; side effects such as external function calls do not always roll back. Execution still requires authorization for the actual target and effects.

### Evidence and Plan Interpretation

Use these checks when interpreting a plan:

- Start with query text, representative parameters, data size/distribution, existing indexes/constraints, and the server version. An anonymized plan is enough for initial advice; avoid broad data extraction.
- Plain `EXPLAIN` estimates a plan without executing the statement, but planning still consumes resources, acquires locks, and can evaluate eligible expressions. It is not a sandbox. Use execution only within the authorized scope, with suitable statement/lock limits and a suitable environment.
- For an authorized representative SELECT, `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` can expose actual row counts, loops, I/O, and spills. Investigate large estimation errors and statistics before assuming an index is missing.
- Per-loop rows/times and inclusive parent costs must not be summed blindly. Planner cost is not milliseconds.
- Cache warmth, parameter values, concurrent workload, prepared generic/custom plans, client transfer, and result size affect observed latency. Compare like conditions; do not promise a universal latency threshold or fixed speedup.
- Sequential scans, bitmap scans, and sorts are legitimate choices. Do not use `enable_seqscan=off` as a production fix.

## 10. Design Rules and Common Mistakes

1. **Start with B-tree** for ordinary `WHERE`, ranges, and `ORDER BY`.
2. **Column order matters:** for frequent multicolumn queries, equality fields usually come first, followed by range/sort fields; check actual plans.
3. **One multicolumn index does not replace every single-column index:** `(user_id, status)` is often a poor fit for frequent `WHERE status = ...`.
4. **Do not duplicate indexes without a reason:** `(user_id, status)` often also serves `WHERE user_id = ...`; a separate `(user_id)` may be redundant, depending on workload.
5. Choose **GIN** for `jsonb`, arrays, and full-text search; **BRIN** for large physically correlated ranges.
6. **Partial** indexes are especially useful for a small, actively used part of a table.
7. **Expression** indexes apply when filtering by an expression (`lower(email)`), not the original field.
8. **`LIKE '%text%'`:** an ordinary B-tree generally does not help; consider `pg_trgm`.
9. **An index is not always faster:** `Seq Scan` may be optimal for a small table or a large fraction of its rows.
10. **On a large live table**, consider `CREATE INDEX CONCURRENTLY` to reduce DML blocking; it cannot run inside an ordinary transaction block and has additional requirements and tradeoffs.
11. **Check `EXPLAIN (ANALYZE, BUFFERS)` within authorized execution scope:** the planner makes the final choice.

**Indexes and constraints:** `PRIMARY KEY` and `UNIQUE` automatically create unique indexes; do not duplicate them. A `FOREIGN KEY` does not create an index on referencing columns. For example, deleting a user may scan all orders to check related `orders(user_id)` rows. Evaluate an index against table size and real operations.

**CONCURRENTLY does not mean “no waiting”:** building makes additional passes, creates load, and can wait for transactions. A failure may leave an `INVALID` index: it is not used for lookup but still incurs write overhead. Check `pg_index.indisvalid`; after resolving the cause, recreate or rebuild the index. A failed unique index can in some cases continue enforcing uniqueness.

`ANALYZE`, `VACUUM`, index creation, and rebuilding are operational actions, not passive diagnosis. Check existing equivalent/prefix and constraint-backed indexes, supported operators and PostgreSQL version, write/storage overhead, build cost, and post-build validity. Do not blindly retry `IF NOT EXISTS` against an invalid index. A generic prepared plan can prevent proof of a partial-index predicate.

### Index Naming Convention

This is a project-specific convention, not a PostgreSQL requirement or a universal rule for this skill. Preserve the target repository's established naming scheme.

- Ordinary index: `IDX_<table>__<column_1>-<column_2>`.
- Unique index: `UIDX_<table>__<column_1>-<column_2>`.
- Table and column names use `snake_case`, as in the database.
- `__` separates the table; `-` separates columns in their declared index order.
- Single-column example: `IDX_orders__user_id`.
- A partial index can have a meaningful purpose-based name: `UIDX_orders__one_active_per_user`.
- Keep names within 63 bytes, PostgreSQL's standard limit; abbreviate long parts meaningfully while preserving name uniqueness within the schema.

Quote these names with double quotes in SQL:

```sql
CREATE INDEX "IDX_orders__user_id-created_at"
ON orders (user_id, created_at);

CREATE UNIQUE INDEX "UIDX_users__email"
ON users (email);

CREATE UNIQUE INDEX "UIDX_orders__one_active_per_user"
ON orders (user_id) WHERE status = 'active';
```

## 11. Big O: Algorithmic Complexity

**Big O** describes how work grows as element count `N` increases, not exact execution time.

| Complexity | Name | Example | For `N = 1000` (illustrative) |
|---|---|---|---:|
| `O(1)` | Constant | Array access by index | 1 |
| `O(log N)` | Logarithmic | Binary search, B-tree | ≈ 10 |
| `O(N)` | Linear | Array traversal | 1,000 |
| `O(N log N)` | Linearithmic | Merge sort | ≈ 10,000 |
| `O(N²)` | Quadratic | Two nested loops | 1,000,000 |
| `O(2^N)` | Exponential | Exhaustive enumeration of subsets | ≈ 10³⁰¹ |

Remember:

- `O(1)`: increasing `N` barely changes the work.
- `O(log N)`: doubling `N` adds approximately one step for a base-2 logarithm.
- `O(N)`: doubling `N` doubles the work.
- `O(N²)`: doubling `N` roughly quadruples the work.

---

### PostgreSQL Documentation

- [Indexes](https://www.postgresql.org/docs/current/indexes.html)
- [Index Types](https://www.postgresql.org/docs/current/indexes-types.html)
- [Multicolumn Indexes](https://www.postgresql.org/docs/current/indexes-multicolumn.html)
- [Combining Multiple Indexes](https://www.postgresql.org/docs/current/indexes-bitmap-scans.html)
- [Partial Indexes](https://www.postgresql.org/docs/current/indexes-partial.html)
- [Index-Only Scans](https://www.postgresql.org/docs/current/indexes-index-only-scans.html)
- [Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
- [Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [CREATE INDEX: Building Indexes Concurrently](https://www.postgresql.org/docs/current/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY)
- [PostgreSQL 18: skip scan](https://www.postgresql.org/docs/18/release-18.html)
