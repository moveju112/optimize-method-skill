# 스택: Node / JS / TS

## 측정
- 쿼리·IO 계획: PG `EXPLAIN ANALYZE`, MySQL `EXPLAIN`. ORM 쿼리 로그로 발급 쿼리 수 카운트.
- 타이밍: `node --prof`(+`--prof-process`), clinic.js (doctor/flame), `console.time`/`performance.now()`.
- 정적: ESLint `no-await-in-loop`, 루프 안 `await query()`, ORM lazy 관계 접근(N+1).

## 안티패턴 (Node 구현)
- 읽기 N+1: DataLoader 배칭, Prisma `include`/`findMany({ where: { id: { in: ids } } })`, TypeORM `relations`.
- 쓰기 N+1: `createMany` / 트랜잭션 배치 insert.
- 마스터 데이터: 모듈 스코프/요청 스코프 Map 메모이즈.
- 무거운 의존성: 모듈 top-level 1회 require, 또는 lazy init 가드.
- 외부 API: 응답 캐시(LRU/Redis) + Cache-Control TTL.

## return 깨짐 함정
- `Array.prototype.sort` 는 ES2019+ 안정 정렬이나 비교함수 누락 시 사전식 정렬(숫자 깨짐).
- 객체 키 순서: 정수형 키는 자동 정렬됨 → JSON 출력 순서 의도와 다를 수 있음.
- batch 결과 순서가 입력 순서와 다름 → id→row Map 후 입력순 재조립.
- float 합산 순서 변경 시 끝자리 차이.

## 검증
- 함수 입출력을 JSON 직렬화해 before/after diff.
- ORM 쿼리 로그로 발급 쿼리 수 N→1 확정 확인.
- 비동기 경합: 반복 호출/동시 호출에서 캐시 오염 없는지.
