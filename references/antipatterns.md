# Anti-pattern catalog (language-neutral archetypes + stack implementation)

Each archetype has 3 fields.
- Signal: what to look for in code to suspect it.
- Transform: how to change it safely.
- Return trap: where this transform commonly breaks the return.

---

## 1. Read N+1 (single-row fetch inside a loop)
- Signal: single-row DB/cache fetch by ID inside a loop body. Call count scales with input size.
- Transform: collect IDs before the loop, fetch in one multi-row query → index into an ID map → loop does only map lookups.
- Return trap: multi-row result order may differ from input order. The original loop preserves input order, so re-assemble in input-ID order.
- Stack: PHP=`IN(...)`+`array_column(rows,null,'Key')` / Node=DataLoader·Prisma `include` / Python=`prefetch_related`·`in_bulk` / Go=pgx batch·`= ANY($1)`.

## 2. Write N+1 (INSERT/UPDATE per loop iteration)
- Signal: execute/save inside a loop.
- Transform: accumulate targets into an array and after the loop do a multi-row INSERT [ON DUPLICATE KEY UPDATE / UPSERT], one round-trip.
- Return trap: ensure upsert semantics match single-row (increment vs overwrite). `col=VALUES(col)` + inline time column preserves equivalence.
- Stack: PHP=multi-row VALUES / Node=`createMany` / Python=`bulk_create` / Go=`pgx.CopyFrom`.

## 3. Duplicate exception → delegate to a constraint
- Signal: try-catch swallowing a duplicate INSERT exception (costly exception flow + race condition).
- Transform: `INSERT IGNORE` / `ON CONFLICT DO NOTHING` lets the engine handle it; remove the try-catch.
- Return trap: confirm other branches of the caught exception did not disappear too.

## 4. Master/design-data re-loaded repeatedly → memoize
- Signal: master data that never changes within a request is re-fetched every loop.
- Transform: store the first fetch in a request-scope cache (member var/map); use a null marker to detect a hit.
- Return trap: if the cache key is order-sensitive, the key order must be preserved.

## 5. Hoist pure computation / multi-step lookup out of the loop
- Signal: same pure function or 2-step array lookup re-computed every loop with the same input.
- Transform: hoist into a local variable once at loop start (e.g. `$unitData = $unitDataList[$uid]`).
- Return trap: confirm the hoisted target is truly loop-invariant (no per-iteration values mixed in).

## 6. Heavy dependency re-loaded repeatedly → lazy init
- Signal: same library loaded per loop/object.
- Transform: null-guard lazy init (`if (x === null) x = load()`).
- Return trap: none (for side-effecting libraries, mind the init timing difference).

## 7. External API response (rarely changes) re-requested every request → cache
- Signal: external HTTP round-trip every request.
- Transform: cache the response (use Cache-Control max-age as TTL where possible); force refresh only on key miss/rotation.
- Return trap: confirm the value change at TTL expiry is within acceptable behavior.

## 8. Duplicate post-processing per branch / per-instance default re-creation
- Signal: both if/else branches call the same post-processing. A default is re-created every count loop.
- Transform: branches set only the input; do common post-processing once outside the branch. Build input-dependent parts (baseUnitData) once; only per-instance values (uuid etc.) in the loop.
- Return trap: do not flatten subtle per-branch differences via the shared path.

---

## Sort/set/float high-risk traps (these break return most often)
- **IN/batch result order**: differs from input. To reproduce the original loop order, re-sort in input-ID order (index by map, re-assemble in input order).
- **Sort stability**: PHP `usort` is stable only on 8.0+, unstable on earlier/some languages → tie rows reorder. If the original relies on stable sort, name an explicit tie-breaker key.
- **Dedup key preservation**: `array_unique` keeps keys; set conversion may lose order/keys. Result changes by whether you re-index (`array_values`).
- **Float summation order**: batch summation may differ in the last bit if accumulation order changes. Integer summation is safe.
- These 4 break byte-identical immediately. Always verify sort/order before and after the transform.
