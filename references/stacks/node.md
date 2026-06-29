# Stack: Node / JS / TS

## Measure
- Query/IO plan: PG `EXPLAIN ANALYZE`, MySQL `EXPLAIN`. Count issued queries via ORM query log.
- Timing: `node --prof`(+`--prof-process`), clinic.js (doctor/flame), `console.time`/`performance.now()`.
- Static: ESLint `no-await-in-loop`, in-loop `await query()`, ORM lazy relation access (N+1).

## Anti-patterns (Node implementation)
- Read N+1: DataLoader batching, Prisma `include`/`findMany({ where: { id: { in: ids } } })`, TypeORM `relations`.
- Write N+1: `createMany` / transactional batch insert.
- Master data: module-scope/request-scope Map memoize.
- Heavy dependency: require once at module top-level, or lazy-init guard.
- External API: response cache (LRU/Redis) + Cache-Control TTL.

## Return-break traps
- `Array.prototype.sort` is stable on ES2019+ but without a comparator it sorts lexicographically (breaks numbers).
- Object key order: integer-like keys are auto-sorted → JSON output order may differ from intent.
- Batch result order differs from input order → id→row Map then re-assemble in input order.
- Last-digit difference if float summation order changes.

## Verify
- Serialize function in/out to JSON and diff before/after.
- Confirm issued query count N→1 via the ORM query log.
- Async race: no cache contamination under repeated/concurrent calls.
