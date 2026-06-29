# OPTIMIZATION — <project> optimization profile

Project-specific knowledge that the `optimize-method` skill reads.
Process lives in the skill; project-specific knowledge lives here (2-layer split).
Copy this file to `docs/OPTIMIZATION.md` and fill in the blanks.

Stack auto-detect result: <stack>  (manifest: <file>)

## 1. Measurement methods
- Query/IO plan: <engine/command, e.g. PG `EXPLAIN ANALYZE` / MySQL `EXPLAIN`>
- Timing/profiler: <e.g. cProfile / clinic / pprof / request log>
- Static/N+1 detection: <e.g. ORM query log, lint rule, grep for in-loop calls>

## 2. Anti-patterns (project-specific)
- <in-loop data access → which batch method>
- <cache layer / master-data location>
- <DB/connection split rule if any (e.g. game DB / account DB)>
- <data-access layer rule: where queries happen (via Model/Repository etc.)>

## 3. Fix rules
- <coding rules doc path>
- <DB access layer rule>
- <existing method signature preservation policy (no delete, add a version, etc.)>

## 4. Verification methods
- return diff means: <serialization comparison / golden file / response compare>
- baseline capture method: <how to call before the fix to dump results>
- state-variety items: <cache hit·miss, permission branch, new/existing, etc.>
- non-deterministic output judgment: <if RNG/time values exist, fixed-seed or distribution comparison criteria>
- The user commits (no auto-commit).
