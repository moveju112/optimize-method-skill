# Report format — candidate table / selection / verification matrix / post-fix tracking

## Candidate table (before the fix, present to the user first)
When you find slow spots, **before fixing** report them as a table and get the user's selection.
The leading `ID` is an auxiliary tracking column.
The required 6 columns (대상 / 느린 이유 / 변경 추천 방향 / 코드 수정 범위 / 예상 속도 향상 / return 영향) are kept verbatim.

| ID | 대상 | 느린 이유 | 변경 추천 방향 | 코드 수정 범위 | 예상 속도 향상 | return 영향 |
|----|------|-----------|----------------|----------------|----------------|-------------|
| C1 | file:line / method | bottleneck evidence (e.g. loop N+1, full scan) | anti-pattern class + 1-line fix | 소/중/대 + line count | 확정분 \| est.추정분 | 없음 |

Table rules.
- **ID**: C1, C2 … sequence. Track all stages (select·fix·verify·report) 1:1 by this ID.
- **대상**: file:line / method.
- **느린 이유**: evidence-based. If not measured, mark `(정적 추정, 측정 미실행)`.
- **변경 추천 방향**: anti-pattern class + concrete 1-line fix.
- **코드 수정 범위**: 소 (within one method) / 중 (add a Model method) / 대 (multiple files·signature) + rough line count.
- **예상 속도 향상**: always 2 parts `확정분 | est.추정분`.
  - fixed = assertable in code (e.g. `DB왕복 12→1`, `쿼리 N→1`).
  - estimate = time %·multiple must be prefixed `est.` (e.g. `est. ~70%↓`). If unmeasurable, `측정 필요`.
- **return 영향**: default `없음`. `있음`/`불확실` are risk candidates. `비결정` for methods with RNG·shuffle·time values.
- **Sort**: descending by priority score.
  - score = (impact ×3) − (scope ×2) − (risk ×5).
  - impact: big (N→1·70%↓+)=3 / mid=2 / small=1.
  - scope: 소=1 / 중=2 / 대=3.
  - risk: 없음=0 / 비결정=1 / 불확실=1 / 있음=2.
  - High-impact·small-scope·safe candidates rise to the top. The score need not be in the table but is the sort basis.

## Getting candidate selection (gate)
- Right after the candidate table, **always** output a one-line query.
  - e.g. `어떤 후보를 진행할까요? (예: C1,C2 / 전체 / 위험제외 / 보류:C4)`
- Do NOT edit before the user picks an ID (gate equal to "no fix without measurement").
- `전체` includes only candidates whose return impact is `없음`/`비결정`.
- `있음`/`불확실` candidates proceed only if the user **explicitly** names that ID.

## Verification matrix (step 4 output)
Rows = selected candidate IDs, columns = the 6 angles. Cells = PASS/FAIL/N/A.

| ID | normal | boundary | empty | sort·keys·count | state-variety | repeated |
|----|--------|----------|-------|-----------------|---------------|----------|
| C1 | PASS | PASS | PASS | PASS | PASS | PASS |

- Any one cell FAIL → that ID is not applied, revert only that change, record as 'rollback' in the post-fix table.
- Non-deterministic methods judge by `시드 고정 후 동일성` or `분포·개수·키집합 동일` instead of bit-identical values. Non-comparable angles get N/A + reason.

## Post-fix report — before→after tracking table
Every selected ID appears as one row (including held/rolled-back with reasons). Omissions become visually obvious.

| ID | 적용여부 | 수정 위치 | 속도 before→after | return 동일성 | 비고 |
|----|----------|-----------|-------------------|---------------|------|
| C1 | 적용 | file:line | fixed measured + (est. hit?) | 6/6 PASS | — |
| C4 | 롤백 | — | — | sort FAIL | tie order changed |

- Speed: fixed (query/round-trip count) is the measured value, estimate (time %) is annotated 'predicted vs actual' hit/miss.
- Do not mix fixed and estimated values.
- If the estimate is badly off (e.g. added index but optimizer did not use it), note the re-measure reason in 비고 and 'no effect → recommend rollback'.

## Progress (update every turn during the fix, one line)
- `C1 검증완료 / C2 수정중 / C3 보류(사용자) / C4 롤백(정렬 FAIL)`
- One candidate = one state. Consistent with the period=newline rule.
