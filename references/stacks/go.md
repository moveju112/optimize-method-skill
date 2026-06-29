# Stack: Go

## Measure
- Query/IO plan: pgx `EXPLAIN ANALYZE`, `database/sql` `DB.Stats()` (connection wait).
- Timing: `pprof`(cpu/mem), `go test -bench -benchmem`, `runtime/trace`.
- Static: in-loop single `QueryRow`, `go vet`, race detector (`-race`).

## Anti-patterns (Go implementation)
- Read N+1: `WHERE id = ANY($1)` (pq.Array/pgx), batch fetch then map index.
- Write N+1: `pgx.CopyFrom`, multi-row VALUES, `Batch`.
- Master data: `sync.Once`/map cache, request-context cache.
- Heavy dependency: `sync.Once` lazy init.

## Return-break traps
- `sort.Slice` is unstable → use `sort.SliceStable` or a tie-breaker to preserve tie order.
- map iteration order is non-deterministic → fix field/key order on JSON marshal (struct field order, sorted keys).
- Batch result order differs from input order → id→struct map then re-assemble in input order.
- Float summation order.

## Verify
- struct → `json.Marshal` then byte compare before/after (assuming fixed field order).
- Count queries via driver hook/log, compare time with `go test -bench`.
