# 스택별 측정·검증 도구 매핑

프로파일이 없을 때 이 표로 측정 수단을 고른다.
3축 = ① 쿼리·IO 계획 / ② 타이밍·프로파일러 / ③ 정적·N+1 탐지. + return diff 수단.

| 스택 | ① 쿼리·IO 계획 | ② 타이밍·프로파일러 | ③ 정적·N+1 탐지 | return diff 수단 |
|------|----------------|----------------------|------------------|------------------|
| PHP/MySQL | `EXPLAIN`, slow query log | 요청 로그 method 타이밍, Xdebug/SPX | 루프 내 Model/loadCache 호출 grep | JSON-RPC result 덤프 후 정규화 비교 |
| Node/JS | PG `EXPLAIN ANALYZE`, MySQL `EXPLAIN` | `node --prof`, clinic.js, `console.time` | ESLint no-await-in-loop, ORM 쿼리 로그(N+1) | 함수 입출력 JSON 직렬화 diff |
| Python | `EXPLAIN ANALYZE`, django-debug-toolbar | `cProfile`, `py-spy`, `timeit` | ORM `select_related`/`prefetch` 부재, 루프 쿼리 | `json.dumps(sort_keys=True)` diff |
| Go | `EXPLAIN ANALYZE`(pgx), `database/sql` stats | `pprof`, `go test -bench`, `-benchmem` | 루프 내 단건 Query 호출, `go vet` | struct→JSON marshal diff (필드순 고정) |
| SQL (엔진중립) | `EXPLAIN` / `EXPLAIN ANALYZE` | DB 서버 slow log | 상관 서브쿼리·함수 인덱스 무력화 | 결과셋 정렬 후 행 단위 비교 |
| Frontend | Network 워터폴 | Performance API, React Profiler | 불필요 리렌더, 리스트 key 누락 | 렌더 출력 스냅샷 비교 |

## 판정 신호 (공통)
- 쿼리 계획에 풀스캔(seq scan / `type=ALL`), 정렬 임시테이블(filesort/temporary), 과다 행 추정 → 즉시 후보.
- 호출 수가 입력 N에 비례 증가 → N+1 강한 신호.
- 같은 입력에 같은 외부 왕복 반복 → 캐시/메모이즈 후보.
