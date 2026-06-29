# 스택: Go

## 측정
- 쿼리·IO 계획: pgx `EXPLAIN ANALYZE`, `database/sql` `DB.Stats()` (커넥션 대기).
- 타이밍: `pprof`(cpu/mem), `go test -bench -benchmem`, `runtime/trace`.
- 정적: 루프 내 단건 `QueryRow`, `go vet`, race detector(`-race`).

## 안티패턴 (Go 구현)
- 읽기 N+1: `WHERE id = ANY($1)` (pq.Array/pgx), 배치 조회 후 map 색인.
- 쓰기 N+1: `pgx.CopyFrom`, 멀티-row VALUES, `Batch`.
- 마스터 데이터: `sync.Once`/맵 캐시, 요청 컨텍스트 캐시.
- 무거운 의존성: `sync.Once` 지연 초기화.

## return 깨짐 함정
- `sort.Slice` 는 불안정 → 동순위 순서 보존 필요하면 `sort.SliceStable` 또는 tie-breaker.
- map 순회 순서는 비결정적 → JSON marshal 시 필드/키 순서 고정(struct 필드 순서, 정렬된 키).
- batch 결과 순서가 입력 순서와 다름 → id→struct map 후 입력순 재조립.
- float 합산 순서.

## 검증
- struct → `json.Marshal` 후 before/after 바이트 비교 (필드 순서 고정 전제).
- 쿼리 수는 드라이버 훅/로그로 카운트, `go test -bench` 로 시간 비교.
