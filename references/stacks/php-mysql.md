# Stack: PHP / MySQL

If a project profile exists, it wins. This file is the PHP/MySQL generic fallback.
Concrete cases (hnote etc.) live in the `docs/OPTIMIZATION.md` profile as the single source.

## Measure
- Query/IO plan: `EXPLAIN <쿼리>`. `type=ALL`(full scan), excess `rows`, `Using filesort/temporary` are signals. slow query log.
- Timing: request-log per-method/endpoint processing time. Xdebug profiler or SPX.
- Static: grep in-loop single DB/cache calls, static-table query every request, in-loop redundant computation.

## Anti-patterns (PHP/MySQL implementation)
- Read N+1: collect IDs before the loop, `IN(...)` multi-row once → `array_column(rows, null, 'Key')` map index → loop does lookups.
- Write N+1: accumulate into array then multi-row `INSERT ... VALUES (...),(...) [ON DUPLICATE KEY UPDATE col=VALUES(col)]`. Inline time column `UNIX_TIMESTAMP()`.
- Ignore duplicate exception: `INSERT IGNORE` instead of try-catch.
- Master data: cache once into a member-var map, detect hit by a null marker.
- DB access layer: queries go inside Model methods. Library goes via `$this->model->method()` / `$this->loadModel('X')` (no direct DB call from Library).
- Existing single-row Model methods are used by other callers, so do not delete. **Add** the IN/bulk version.

## Return-break traps (PHP-specific)
- Assoc-array key order: both `===` and JSON object are order-sensitive. When merging IN, re-assemble result in input-ID order.
- int vs string: `'5'` vs `5` differ in JSON → fail.
- `usort`: stable sort only on PHP 8.0+. Need a tie-breaker key to preserve tie order.
- IN multi-row result: if the contract returns in input order, position mapping (`array_combine`) is safe; otherwise index by key then re-assemble in input order.

## Verify
- Re-run `EXPLAIN` on the changed query (did it hit the index) + confirm query/round-trip count dropped.
- Dump JSON-RPC `result` before and after, normalized compare (`references/verification.md`).
- Call with several data-state accounts (new/existing, cache hit/miss, permission branch) and compare.
