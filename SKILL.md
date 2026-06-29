---
name: optimize-method
version: "2.0.0"
description: Use when optimizing a slow method/function/query for speed — triggered by "X 속도 개선", "느린 메소드 최적화", "성능 개선", "이거 왜 느려", "최적화해줘", "N+1 제거", "쿼리 줄여줘", "루프 안 쿼리", "벌크로 묶어", "캐시 적용해서 빠르게", or any request to make existing code faster without changing its behavior. ⛔ Return value MUST stay byte-identical. Project-agnostic 4 steps (measure→classify→fix→verify); reads project-local profile if present, else auto-detects stack. Not this skill if behavior/spec changes. Log analysis only → request-log-tracer.
---

# optimize-method — slow code optimization (generic)

Keep behavior identical; improve only speed.
Process is shared across all projects; knowledge comes from the profile/stack files.
Respond to the user in Korean (skill output stays Korean).

## ⛔ Top rule — final return value is immutable (SSOT)
- **A method's final return value MUST NEVER change.**
- Same input → return MUST be 100% identical before and after the fix.
- Identical = value, type, keys, order, count all match.
- If even 1 bit differs, the optimization failed.
- On failure, do NOT apply; report instead.
- If identity cannot be proven, classify as "risk" and tell the user first.
- This rule applies to every step below (steps only reference it as `(철칙 적용)`).

## 0. Load project profile (first)
- Look for project-specific knowledge in this order.
  1. `docs/OPTIMIZATION.md`
  2. `docs-local/OPTIMIZATION.md`
  3. root `.optimize-method.md` or `OPTIMIZATION.md`
- If a profile exists, use its measure/anti-pattern/verify commands first.
- If no profile → fall back to stack auto-detection.
  - Assert the stack in one line from the root manifest.
  - `composer.json`→PHP, `package.json`→Node, `pyproject.toml`/`requirements.txt`→Python, `go.mod`→Go, `*.csproj`→.NET, `Gemfile`→Ruby, `pom.xml`/`build.gradle`→JVM.
  - If more than one, narrow by the target file's extension.
  - Read `references/stacks/{stack}.md` for the detected stack to fill in measure/anti-pattern/verify.
- If there is no profile and this project will be optimized repeatedly, propose drafting `docs/OPTIMIZATION.md` once from `references/profile-template.md` (not forced).

## 1. Measure (gather evidence)
- **Never fix on a guess.** Find evidence of what is slow first.
- Measure on 3 axes. Use stack-specific commands from the step 0 profile/stack file.
  - ① Query/IO plan: check full scans, index misses, excess rows via the execution plan (e.g. `EXPLAIN`, slow query log).
  - ② Request/exec timing: pinpoint slow spots by per-endpoint/function processing time.
  - ③ Static analysis: hunt in-loop calls, redundant computation, N+1, missing cache directly in code.
- When no measurement tooling exists (no log/profiler/repro-data access):
  - Proceed with static analysis only, but mark the candidate table's "reason" with `(정적 추정, 측정 미실행)`.
  - For "expected speedup", quantify only fixed values (query/call count N→1); mark time as `측정 필요`.
  - Ask the user in one line for measurement tooling (log/profile access) to get accurate numbers.
  - If baseline capture is impossible, route to the UNPROVEN gate in step 4.
- Output: a one-line assertion of "which line/query is what % of total time".

## 2. Classify cause (anti-pattern mapping)
- Map the observed bottleneck to a known pattern.
- 6 core archetypes: N+1 (read/write), missing cache, in-loop redundant computation, inefficient sort/ranking, full fetch, index miss.
- See `references/antipatterns.md` for each archetype's detection signal, safe transform, return-breaking trap, and per-stack implementation.
- Apply project-specific anti-patterns from the profile/stack file too.

## 3. Fix (minimally invasive)
- Do not change behavior (rule applies).
- Start with the single biggest bottleneck. One change at a time.
- Changing the RNG/time source itself is forbidden (behavior change).
- Follow project coding rules (CLAUDE.md). The skill does not override them.

## 4. Verify (gate) — multi-angle mandatory
- **Read and follow** `references/verification.md` exactly for the procedure and comparison criteria.
- Core skeleton:
  - Capture the baseline **before** the fix (save it). Baseline cannot be made after the fix.
  - Fix → re-call with the same inputs → diff after normalization.
  - Multi-angle cases: normal / boundary (0·1·bulk, first·last) / empty (empty array·null·missing ID) / sort·keys·count / state (cache hit·miss, new·existing) / repeated calls.
- If even one case's return differs, do not apply (rule applies).
- For non-deterministic output (RNG·shuffle·time·float), switch from raw diff to invariant checks (`references/verification.md`).
- Output: a candidate-ID × 6-angle verification matrix (`references/reporting.md`).

## Apply/rollback gate
- 3-way verdict.
  - PASS (all cases byte-identical or invariant preserved) → keep applied.
  - FAIL (any case diffs) → revert only that change with `git checkout -- <file>`, then report.
  - UNPROVEN (no repro data/access to capture baseline) → do NOT apply code, state "human verification needed", report.
- If only some of several candidates FAIL, revert only the FAIL candidates individually (pre-commit, so identify by hunk via `git diff`).
- Revert is a working-tree operation. The user commits (no auto-commit).

## Report format
- Follow `references/reporting.md` for the candidate table, selection gate, verification matrix, and post-fix tracking table.
- Keep the candidate table's 6 columns verbatim: 대상 / 느린 이유 / 변경 추천 방향 / 코드 수정 범위 / 예상 속도 향상 / return 영향.
- Right after presenting the candidate table, always output the selection query, and do not edit before the user picks an ID (gate equal to the top rule).

## References (load only when needed)
- `references/antipatterns.md` — anti-pattern archetypes (signal/transform/trap) + per-stack implementation + sort·order·float high-risk traps.
- `references/verification.md` — baseline capture procedure, identity comparison criteria, non-deterministic branch, multi-angle cases. The single source of truth that step 4 points to.
- `references/reporting.md` — candidate table (6 columns + ID), selection gate, priority score, verification matrix, before→after tracking table.
- `references/stacks/_matrix.md` — stack × measurement-tool map. Use first when no profile.
- `references/stacks/<stack>.md` — php-mysql / node / python / go / sql / frontend per-stack measure/anti-pattern/verify.
- `references/profile-template.md` — blank profile form for a new project.

## What this skill does NOT do
- Guess-based fixes without measurement.
- Refactoring that changes behavior (that is a different task).
- Changing the RNG/time source.
- Commit (the user does it).
