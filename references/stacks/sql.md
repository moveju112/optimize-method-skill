# Stack: SQL (engine-neutral)

Execution-plan reading and query optimization knowledge common across DB engines.

## Measure
- `EXPLAIN` (plan only) / `EXPLAIN ANALYZE` (actual run time·rows). Supported by PG/MySQL 8+/SQLite.
- Collect slow queries from the server slow query log.

## How to read (warning signals)
- Full scan: PG `Seq Scan`, MySQL `type=ALL`. On a big table, an index candidate.
- Sort cost: PG `Sort` + disk, MySQL `Using filesort`/`Using temporary`.
- Excess row estimate or actual≫estimate → update statistics (ANALYZE) or review the index.
- Correlated subquery (re-run per row) → flatten with JOIN/window function.
- Index disabled: function on column (`WHERE DATE(col)=...`), leading wildcard `LIKE '%x'`, implicit cast from type mismatch.

## Safe transforms
- N+1 → `WHERE id IN (...)` / `= ANY(...)` at once.
- SELECT only needed columns (avoid `SELECT *`), only needed rows (`LIMIT`/proper WHERE).
- Multi-row INSERT, UPSERT (`ON CONFLICT`/`ON DUPLICATE KEY UPDATE`).

## Return (result set) break traps
- A query without `ORDER BY` has no guaranteed order → even if order changes from the optimization it may not be 'identical'. State the order the original relied on.
- Row order after `GROUP BY`/`DISTINCT`, and `IN` result order, are unrelated to input order.
- NULL sort position (NULLS FIRST/LAST) differs by engine/version.

## Verify
- Re-run `EXPLAIN ANALYZE` on the changed query (confirm the plan improved).
- Compare row-by-row after sorting the result set by the same key.
