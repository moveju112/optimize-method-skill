# optimize-method

A Claude Code skill that makes a slow method/query faster **while keeping behavior identical**.

> ⛔ Top rule: the final return value MUST stay byte-identical.
> Same input → return is 100% identical before and after (value·type·keys·order·count).
> If even 1 bit differs, do not apply — report instead.

## What it does

- Fires on requests like "X 속도 개선", "느린 메소드 최적화", "N+1 제거", "쿼리 줄여줘".
- Runs 4 steps: measure → classify cause → fix → verify.
- **Before** fixing, reports slow candidates as a table and gets the user's selection.
- **After** fixing, verifies return identity from many angles (normal/boundary/empty/sort/state/repeat).
- Skill instructions are in English (token-efficient); user-facing output stays Korean.

## Install

Clone this repo into the Claude Code skills directory.

```bash
git clone https://github.com/moveju112/optimize-method-skill.git \
  ~/.claude/skills/optimize-method
```

Done once `~/.claude/skills/optimize-method/SKILL.md` is recognized.

## Structure (2 layers)

Process (generic) and knowledge (per-project) are split.

| Layer | Location | Role |
|-------|----------|------|
| Process | `SKILL.md` + `references/` | shared across all projects |
| Knowledge | the project's `docs/OPTIMIZATION.md` | that project's measure/anti-pattern/verify |

If a project has no profile, it auto-detects the stack from the manifest and falls back to
`references/stacks/<stack>.md` (php-mysql / node / python / go / sql / frontend).

For a new project, copy `references/profile-template.md` to `docs/OPTIMIZATION.md` and fill it in.

## Files

```
SKILL.md                          slim orchestrator (triggers·top rule·4 steps·gate)
references/
  antipatterns.md                 8 anti-pattern archetypes + per-stack impl + sort·order·float traps
  verification.md                 baseline capture·identity compare·non-deterministic branch·multi-angle cases
  reporting.md                    candidate table (6 cols)·selection gate·priority·verification matrix
  profile-template.md             blank profile form for a new project
  stacks/
    _matrix.md                    stack × measurement-tool map
    php-mysql.md  node.md  python.md  go.md  sql.md  frontend.md
```

## Verification principle

- Capture the baseline **before** the fix (it cannot be made afterward).
- Fix → re-call with the same inputs and controlled state → compare original serialized bytes.
- 4-way verdict: PASS (tested cases byte-identical) / FAIL (controlled difference) / INVARIANTS_ONLY (baseline and completed invariant checks only) / UNPROVEN (missing baseline/required comparison evidence).
- Only PASS retains the optimization. Invariant-only checks never prove identity; deterministic float rounding differences fail. Restore only task-owned edits and preserve other work.
- The user commits (the skill never auto-commits).
