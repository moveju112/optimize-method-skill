# 스택: Python

## 측정
- 쿼리·IO 계획: PG/MySQL `EXPLAIN ANALYZE`, django-debug-toolbar SQL 패널, SQLAlchemy echo.
- 타이밍: `cProfile`(+`pstats`), `py-spy`(샘플링), `timeit`(마이크로벤치).
- 정적: ORM 루프 내 쿼리, `select_related`/`prefetch_related` 부재, 리스트 컴프 내 쿼리.

## 안티패턴 (Python 구현)
- 읽기 N+1: Django `select_related`(FK join)/`prefetch_related`(역참조), `in_bulk(ids)`, SQLAlchemy `selectinload`.
- 쓰기 N+1: `bulk_create` / `bulk_update`, `executemany`.
- 마스터 데이터: `functools.lru_cache`, 요청 스코프 dict 메모이즈.
- 외부 API: `requests` 세션 재사용 + 응답 캐시.

## return 깨짐 함정
- `sorted`/`list.sort` 는 안정 정렬(보존 OK)이나 `key` 누락 시 비교 규칙 차이.
- dict 키 순서: 3.7+ 삽입 순서 보존 → batch 재구성 시 순서 보존 신경.
- batch 결과 순서가 입력 순서와 다름 → id→obj dict 후 입력순 재조립.
- float 합산 순서, `Decimal` vs `float` 혼용 주의.

## 검증
- `json.dumps(obj, sort_keys=True, default=str)` 로 정규화 후 before/after diff (단, 키 순서 계약이 있으면 sort_keys 빼고 원순서 비교).
- 쿼리 수: `len(connection.queries)` 또는 assertNumQueries 로 N→1 확정.
