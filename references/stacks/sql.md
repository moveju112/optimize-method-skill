# 스택: SQL (엔진 중립)

DB 종류와 무관한 실행계획 판독·쿼리 최적화 공통 지식.

## 측정
- `EXPLAIN` (계획만) / `EXPLAIN ANALYZE` (실제 실행 시간·행수). PG/MySQL 8+/SQLite 지원.
- 서버 slow query log 로 느린 쿼리 수집.

## 읽는 법 (위험 신호)
- 풀스캔: PG `Seq Scan`, MySQL `type=ALL`. 큰 테이블이면 인덱스 후보.
- 정렬 비용: `Sort` + 디스크, MySQL `Using filesort`/`Using temporary`.
- 행 추정 과다 또는 실제≫추정 → 통계 갱신(ANALYZE) 또는 인덱스 검토.
- 상관 서브쿼리(행마다 재실행) → JOIN/윈도우 함수로 평탄화.
- 인덱스 무력화: 컬럼에 함수 적용(`WHERE DATE(col)=...`), 선두 와일드카드 `LIKE '%x'`, 타입 불일치 암묵 캐스팅.

## 안전 변환
- N+1 → `WHERE id IN (...)` / `= ANY(...)` 한 번에.
- 필요한 컬럼만 SELECT (`SELECT *` 회피), 필요한 행만 (`LIMIT`/적절한 WHERE).
- 멀티-row INSERT, UPSERT(`ON CONFLICT`/`ON DUPLICATE KEY UPDATE`).

## return(결과셋) 깨짐 함정
- `ORDER BY` 없는 쿼리는 순서 미보장 → 최적화로 순서가 바뀌어도 '동일'이 아닐 수 있음. 원본이 의존하던 순서를 명시.
- `GROUP BY`/`DISTINCT` 후 행 순서, `IN` 결과 순서는 입력 순서와 무관.
- NULL 정렬 위치(NULLS FIRST/LAST)가 엔진/버전 따라 다름.

## 검증
- 수정 쿼리 `EXPLAIN ANALYZE` 재실행 (계획 개선 확인).
- 결과셋을 동일 키로 정렬 후 행 단위 비교.
