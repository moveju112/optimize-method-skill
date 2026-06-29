# 스택: PHP / MySQL

프로젝트 프로파일이 있으면 그쪽이 우선. 이 파일은 PHP/MySQL 일반형 fallback.
hnote 등 구체 사례는 `docs/OPTIMIZATION.md` 프로파일이 단일 소스.

## 측정
- 쿼리·IO 계획: `EXPLAIN <쿼리>`. `type=ALL`(풀스캔), `rows` 과다, `Using filesort/temporary` 가 신호. slow query log.
- 타이밍: 요청 로그 method/엔드포인트별 처리시간. Xdebug profiler 또는 SPX.
- 정적: 루프 안 DB/캐시 단건 호출, 매 요청 정적 테이블 쿼리, 루프 내 중복 계산을 grep.

## 안티패턴 (PHP/MySQL 구현)
- 읽기 N+1: 루프 전 ID 모아 `IN(...)` 다건 1회 → `array_column(rows, null, 'Key')` 맵 색인 → 루프는 lookup.
- 쓰기 N+1: 배열 누적 후 멀티-row `INSERT ... VALUES (...),(...) [ON DUPLICATE KEY UPDATE col=VALUES(col)]`. 시간컬럼 `UNIX_TIMESTAMP()` 인라인.
- 중복 예외 무시: try-catch 대신 `INSERT IGNORE`.
- 마스터 데이터: 멤버 변수 맵에 1회 캐시, null 마커로 적중 판단.
- DB 접근 계층: 쿼리는 Model 메서드 안에서. Library는 `$this->model->method()` / `$this->loadModel('X')` 경유 (Library DB 직접 호출 금지).
- 기존 단건 Model 메서드는 다른 호출자가 쓰므로 삭제 금지. IN/벌크 버전을 **추가**한다.

## return 깨짐 함정 (PHP 특유)
- 연관배열 키 순서: `===`·JSON object 모두 순서 민감. IN 합칠 때 결과를 입력 ID 순으로 재조립.
- int vs string: `'5'` vs `5` 는 JSON에서 다름 → 실패.
- `usort`: PHP 8.0+만 안정 정렬. 동순위 순서 보존 필요하면 tie-breaker 키.
- IN 다건 결과는 입력 순서대로 반환되는 계약이면 위치 매핑(`array_combine`)이 안전, 아니면 키로 색인 후 입력순 재조립.

## 검증
- 수정한 쿼리 `EXPLAIN` 재실행 (인덱스 탐는지) + 쿼리/왕복 호출 수 감소 확인.
- JSON-RPC `result` 를 수정 전후 덤프해 정규화 비교 (`references/verification.md`).
- 여러 데이터 상태 계정(신규/기존, 캐시 hit/miss, 권한 분기)으로 호출해 대조.
