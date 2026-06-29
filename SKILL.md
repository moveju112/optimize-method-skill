---
name: optimize-method
version: "2.0.0"
description: Use when optimizing a slow method/function/query for speed — triggered by "X 속도 개선", "느린 메소드 최적화", "성능 개선", "이거 왜 느려", "최적화해줘", "N+1 제거", "쿼리 줄여줘", "루프 안 쿼리", "벌크로 묶어", "캐시 적용해서 빠르게", or any request to make existing code faster without changing its behavior. ⛔ Return value MUST stay byte-identical. Project-agnostic 4-step (측정→원인분류→수정→검증); reads project-local profile if present, else auto-detects stack. 동작/스펙 변경을 동반하면 본 스킬 아님. 로그 분석만이면 request-log-tracer.
---

# optimize-method — 느린 코드 최적화 (범용)

동작은 그대로 두고 속도만 개선한다.
절차는 모든 프로젝트 공통, 지식은 프로파일·스택 파일에서 읽는다.

## ⛔ 최상위 철칙 — 최종 return 값 불변 (SSOT)
- **메서드의 최종 return 값은 절대 바뀌면 안 된다.**
- 같은 입력 → 수정 전후 return이 100% 동일해야 한다.
- 동일 = 값·타입·키·순서·개수 전부 일치.
- 1비트라도 다르면 최적화 실패다.
- 실패면 적용하지 말고 보고한다.
- 동일성을 증명 못 하면 "위험"으로 분류해 사용자에게 먼저 알린다.
- 이 철칙은 아래 모든 단계에 적용된다 (각 단계는 `(철칙 적용)`으로만 참조한다).

## 0. 프로젝트 프로파일 로드 (먼저)
- 다음 순서로 프로젝트 전용 지식을 찾는다.
  1. `docs/OPTIMIZATION.md`
  2. `docs-local/OPTIMIZATION.md`
  3. 루트 `.optimize-method.md` 또는 `OPTIMIZATION.md`
- 프로파일이 있으면 그 안의 측정·안티패턴·검증 명령을 최우선으로 쓴다.
- 프로파일이 없으면 → 스택 자동감지로 fallback.
  - 루트 manifest로 스택을 1줄 단정한다.
  - `composer.json`→PHP, `package.json`→Node, `pyproject.toml`/`requirements.txt`→Python, `go.mod`→Go, `*.csproj`→.NET, `Gemfile`→Ruby, `pom.xml`/`build.gradle`→JVM.
  - 둘 이상이면 최적화 대상 파일의 확장자로 좁힌다.
  - 감지된 스택의 `references/stacks/{stack}.md` 를 읽어 측정·안티패턴·검증을 채운다.
- 프로파일이 없고 최적화를 반복할 프로젝트면, `references/profile-template.md` 로 `docs/OPTIMIZATION.md` 초안 작성을 1회 제안한다 (강제 아님).

## 1. 측정 (근거 확보)
- **추측으로 고치지 않는다.** 어디가 느린지 근거부터 잡는다.
- 측정은 3축으로 본다. 스택별 구체 명령은 0단계 프로파일/스택 파일을 따른다.
  - ① 쿼리·IO 계획: 실행계획으로 풀스캔·인덱스 미스·과다 행 확인 (예: `EXPLAIN`, slow query log).
  - ② 요청·실행 타이밍: 엔드포인트/함수 단위 처리시간으로 느린 지점 특정.
  - ③ 정적 분석: 루프 안 호출·중복 계산·N+1·캐시 미사용을 코드에서 직접 탐색.
- 측정 수단이 없을 때 (로그·프로파일러·재현데이터 접근 불가):
  - 정적 분석만으로 진행하되, 후보표 '느린 이유'에 `(정적 추정, 측정 미실행)` 표기.
  - '예상 속도 향상'은 확정값(쿼리/호출 수 N→1)만 정량, 시간은 `측정 필요`.
  - 사용자에게 '측정 수단(로그/프로파일 권한)을 주면 정확해진다'를 1줄 요청.
  - baseline 캡처가 불가하면 4단계의 UNPROVEN 게이트로 보낸다.
- 산출물: "어느 줄/어느 쿼리가 전체 시간의 몇 %인가" 1줄 단정.

## 2. 원인 분류 (안티패턴 매핑)
- 관찰된 병목을 알려진 패턴에 매핑한다.
- 핵심 6원형: N+1(읽기/쓰기), 캐시 미사용, 루프 안 중복 계산, 정렬/랭킹 비효율, 전체 조회, 인덱스 미스.
- 각 원형의 탐지신호·안전 변환·return 깨짐 함정·스택별 구현은 `references/antipatterns.md` 참조.
- 프로파일/스택 파일에 전용 안티패턴이 있으면 추가 적용한다.

## 3. 수정 (최소 침습)
- 동작을 바꾸지 않는다 (철칙 적용).
- 가장 큰 병목 1개부터. 한 번에 한 가지 변경.
- 난수·시간 소스 자체를 바꾸는 변경은 금지 (동작 변경).
- 프로젝트 코딩 규칙(CLAUDE.md)을 따른다. 스킬이 규칙을 덮어쓰지 않는다.

## 4. 검증 (게이트) — 다방면 필수
- 검증 절차와 비교 기준은 `references/verification.md` 를 **반드시 읽고 그대로** 수행한다.
- 핵심 골격:
  - 수정 **전에** baseline을 먼저 캡처(저장)한다. baseline은 수정 후엔 만들 수 없다.
  - 수정 → 동일 입력 재호출 → 정규화 후 diff.
  - 다방면 케이스: 정상 / 경계(0·1·대량, 첫·끝) / 빈값(빈배열·null·미존재 ID) / 정렬·키·개수 / 상태(캐시 hit·miss, 신규·기존) / 반복 호출.
- 한 케이스라도 return이 다르면 적용하지 않는다 (철칙 적용).
- 비결정 출력(RNG·셔플·시간·float)은 raw diff 대신 불변식 검증으로 전환 (`references/verification.md`).
- 산출물: 후보ID × 6각도 검증 매트릭스 (`references/reporting.md`).

## 적용·롤백 게이트
- 판정 3분기.
  - PASS (전 케이스 byte-identical 또는 불변식 보존) → 적용 유지.
  - FAIL (1케이스라도 diff) → 해당 변경만 `git checkout -- <file>` 로 revert 후 보고.
  - UNPROVEN (재현 데이터·권한 없어 baseline 캡처 불가) → 코드 적용 안 함, '사람 검증 필요' 명시 후 보고.
- 후보 여러 개 중 일부만 FAIL이면 FAIL 후보만 개별 revert (커밋 전이므로 hunk 단위 `git diff`로 식별).
- revert는 working tree 조작이다. 커밋은 사용자가 직접 (자동 커밋 금지).

## 보고 형식
- 후보표·선택 게이트·검증 매트릭스·수정 후 추적표 규약은 `references/reporting.md` 를 따른다.
- 후보표 6컬럼은 글자 그대로 유지한다: 대상 / 느린 이유 / 변경 추천 방향 / 코드 수정 범위 / 예상 속도 향상 / return 영향.
- 후보표 제시 직후 반드시 선택 질의를 출력하고, 사용자가 ID를 고르기 전에는 편집하지 않는다 (철칙 동급 게이트).

## 참고 자료 (필요할 때만 로드)
- `references/antipatterns.md` — 안티패턴 원형(탐지신호/변환/함정) + 스택별 구현 + 정렬·순서·float 고위험 함정.
- `references/verification.md` — baseline 캡처 절차·동일성 비교 기준·비결정 분기·다방면 케이스. 4단계가 가리키는 단일 진실원천.
- `references/reporting.md` — 후보표(6컬럼+ID)·선택 게이트·우선순위 점수·검증 매트릭스·before→after 추적표.
- `references/stacks/_matrix.md` — 스택×측정도구 매핑표. 프로파일 없을 때 먼저.
- `references/stacks/<stack>.md` — php-mysql / node / python / go / sql / frontend 스택별 측정·안티패턴·검증.
- `references/profile-template.md` — 새 프로젝트 프로파일 빈 양식.

## 안 하는 것
- 측정 없는 추측성 수정.
- 동작 변경을 동반한 리팩토링 (그건 다른 작업).
- 난수·시간 소스 변경.
- 커밋 (사용자가 직접).
