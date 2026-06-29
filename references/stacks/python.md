# Stack: Python

## Measure
- Query/IO plan: PG/MySQL `EXPLAIN ANALYZE`, django-debug-toolbar SQL panel, SQLAlchemy echo.
- Timing: `cProfile`(+`pstats`), `py-spy`(sampling), `timeit`(microbench).
- Static: in-loop ORM query, `select_related`/`prefetch_related` absence, query inside list comprehension.

## Anti-patterns (Python implementation)
- Read N+1: Django `select_related`(FK join)/`prefetch_related`(reverse), `in_bulk(ids)`, SQLAlchemy `selectinload`.
- Write N+1: `bulk_create` / `bulk_update`, `executemany`.
- Master data: `functools.lru_cache`, request-scope dict memoize.
- External API: reuse `requests` session + response cache.

## Return-break traps
- `sorted`/`list.sort` is stable (preserves OK) but a missing `key` changes comparison rules.
- dict key order: insertion order preserved on 3.7+ → mind order when re-building a batch.
- Batch result order differs from input order → id→obj dict then re-assemble in input order.
- Mind float summation order, `Decimal` vs `float` mix.

## Verify
- Normalize with `json.dumps(obj, sort_keys=True, default=str)` then diff before/after (but if there is a key-order contract, drop sort_keys and compare original order).
- Query count: confirm N→1 via `len(connection.queries)` or assertNumQueries.
