# Per-stack measure/verify tool map

When there is no profile, pick measurement tooling from this table.
3 axes = ① query/IO plan / ② timing/profiler / ③ static/N+1 detection. + return diff means.

| Stack | ① query/IO plan | ② timing/profiler | ③ static/N+1 detection | return diff means |
|-------|-----------------|-------------------|------------------------|-------------------|
| PHP/MySQL | `EXPLAIN`, slow query log | request-log method timing, Xdebug/SPX | grep in-loop Model/loadCache calls | dump JSON-RPC result then normalized compare |
| Node/JS | PG `EXPLAIN ANALYZE`, MySQL `EXPLAIN` | `node --prof`, clinic.js, `console.time` | ESLint no-await-in-loop, ORM query log (N+1) | function in/out JSON serialization diff |
| Python | `EXPLAIN ANALYZE`, django-debug-toolbar | `cProfile`, `py-spy`, `timeit` | ORM `select_related`/`prefetch` absence, in-loop query | `json.dumps(sort_keys=True)` diff |
| Go | `EXPLAIN ANALYZE`(pgx), `database/sql` stats | `pprof`, `go test -bench`, `-benchmem` | in-loop single `Query` call, `go vet` | struct→JSON marshal diff (fixed field order) |
| SQL (engine-neutral) | `EXPLAIN` / `EXPLAIN ANALYZE` | DB server slow log | correlated subquery·function disabling index | compare rows after sorting the result set |
| Frontend | Network waterfall | Performance API, React Profiler | unneeded re-render, missing list key | render output snapshot compare |

## Verdict signals (common)
- Query plan shows full scan (seq scan / `type=ALL`), sort temp table (filesort/temporary), excess row estimate → immediate candidate.
- Call count grows with input N → strong N+1 signal.
- Same external round-trip repeated for same input → cache/memoize candidate.
