# optimize-method

느린 메서드·쿼리를 **동작은 그대로 두고 속도만** 개선하는 Claude Code 스킬.

> ⛔ 최상위 철칙: 최종 return 값은 byte 단위로 동일해야 한다.
> 같은 입력 → 수정 전후 return 100% 일치 (값·타입·키·순서·개수).
> 1비트라도 다르면 적용하지 않고 보고한다.

## 무엇을 하나

- "X 속도 개선", "느린 메소드 최적화", "N+1 제거", "쿼리 줄여줘" 같은 요청에 발동.
- 측정 → 원인분류 → 수정 → 검증 4단계로 진행.
- 고치기 **전에** 느린 후보를 표로 보고하고 사용자 선택을 받는다.
- 고친 **후에** 여러 각도(정상/경계/빈값/정렬/상태/반복)로 return 동일성을 검증한다.

## 설치

이 repo를 Claude Code 스킬 디렉토리에 클론한다.

```bash
git clone https://github.com/moveju112/optimize-method-skill.git \
  ~/.claude/skills/optimize-method
```

`~/.claude/skills/optimize-method/SKILL.md` 가 인식되면 끝.

## 구조 (2계층)

절차(범용)와 지식(프로젝트별)을 분리한다.

| 계층 | 위치 | 역할 |
|------|------|------|
| 절차 | `SKILL.md` + `references/` | 모든 프로젝트 공통 |
| 지식 | 프로젝트의 `docs/OPTIMIZATION.md` | 그 프로젝트 측정·안티패턴·검증 |

프로젝트에 프로파일이 없으면 manifest로 스택을 자동 감지해
`references/stacks/<stack>.md` (php-mysql / node / python / go / sql / frontend)로 fallback 한다.

새 프로젝트는 `references/profile-template.md` 를 `docs/OPTIMIZATION.md` 로 복사해 채우면 된다.

## 파일

```
SKILL.md                          슬림 오케스트레이터 (트리거·철칙·4단계·게이트)
references/
  antipatterns.md                 안티패턴 원형 8종 + 스택별 구현 + 정렬·순서·float 함정
  verification.md                 baseline 캡처·동일성 비교·비결정 분기·다방면 케이스
  reporting.md                    후보표(6컬럼)·선택 게이트·우선순위·검증 매트릭스
  profile-template.md             새 프로젝트 프로파일 빈 양식
  stacks/
    _matrix.md                    스택 × 측정도구 매핑표
    php-mysql.md  node.md  python.md  go.md  sql.md  frontend.md
```

## 검증 원칙

- 수정 **전에** baseline을 먼저 캡처한다 (수정 후엔 만들 수 없다).
- 수정 → 동일 입력 재호출 → 정규화 후 diff.
- 판정 3분기: PASS(적용) / FAIL(해당 변경만 revert) / UNPROVEN(재현 불가 → 사람 검증 필요).
- 커밋은 사용자가 직접 한다 (스킬은 자동 커밋하지 않는다).
