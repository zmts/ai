# Plans, statistics and indexes

- Start with query text, representative parameters, data size/distribution, existing indexes/constraints and version. An anonymized plan is enough for initial advice; avoid broad data extraction.
- Plain EXPLAIN estimates a plan without executing the statement, but planning still consumes resources/acquires locks and can evaluate eligible expressions. EXPLAIN ANALYZE really executes. Use it only within approved execution scope and a suitable environment, with appropriate statement/lock limits.
- For an authorized representative SELECT, EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) can reveal actual rows, loops, IO and spills. Investigate large estimate errors and statistics before assuming a missing index. Per-loop rows/times and inclusive parent costs must not be summed blindly.
- Planner cost is not milliseconds. Cache warmth, parameters, concurrent workload, prepared generic/custom plans, client transfer and result size affect observed latency. Do not promise a universal 100ms target or a fixed speedup.
- Sequential scan can be best for small tables or broad reads. Bitmap scans/sorts are legitimate. Do not set enable_seqscan=off as a production fix.
- Choose B-tree/GIN/GiST/BRIN and expression/composite indexes for actual operators, selectivity, ordering and correlation. Leading columns matter, but available strategies vary by version; avoid absolute claims that later columns can never help.
- Partial index predicates must be provably implied by the query; parameterized generic plans can prevent that proof. INCLUDE can enable coverage, but index-only execution also depends on visibility and may still fetch heap pages.
- Check existing equivalent/prefix indexes and constraint-backed indexes before adding one. Foreign-key referencing columns are not automatically indexed by PostgreSQL; evaluate lookup/delete/update patterns rather than always adding an index.
- ANALYZE/VACUUM/index creation are operational actions, not passive diagnosis. Recommend them with evidence; execute only within scope. Check write overhead, storage, build cost and follow-up validity.

Sources: [EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html), [indexes](https://www.postgresql.org/docs/current/indexes.html), [partial indexes](https://www.postgresql.org/docs/current/indexes-partial.html).
