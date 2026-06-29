# Verification recipe — return identity (differential testing, stack-neutral)

The single source of truth that SKILL step 4 points to.
Prove return is unchanged by "capture then diff", not by guessing.

## 1. Baseline capture (mandatory, before the fix)
- Use the case-ID list fixed in the candidate table as the input set (covers the §4 multi-angle cases).
- **Before** inserting the fix, in the unmodified state, call each case and dump the result to a golden file.
  - Path: `<scratchpad>/golden/<method>__<caseId>.before.json`
  - Capture unit = the serialization boundary the client actually receives (e.g. JSON-RPC `result`).
- Baseline cannot be made after the fix. If you cannot capture it, go to the SKILL "apply/rollback gate" UNPROVEN.
- No measurement/repro data → no baseline → UNPROVEN.

## 2. Fix → after capture
- Call again with the same input set and dump `<method>__<caseId>.after.json`.

## 3. Compare (definition of identity)
- Default gate = is the serialized result byte-identical. This is the truth because it is what the client receives.
- Normalization is a "comparison method", not "introducing tolerance". Type/keys/count must be unconditionally identical.
- If they differ, classify the trap with the table below (all count as "fail", do not let them pass).

| Trap | Symptom | Verdict |
|------|---------|---------|
| Assoc-array key order | JSON object key order changed | fail (preserve order) |
| int/string mix | `5` vs `"5"` | fail |
| null vs missing key | `"k":null` vs no key | fail |
| IN/batch order | result differs from input order | fail (re-sort in input order) |
| float precision | summation order changed, last digit differs | go to non-deterministic branch (4) |

- State the comparison level: string comparison after serialization is the default. PHP array `===` comparison is sensitive to key order/type, so use it only as an aux.

## 4. Non-deterministic output — when raw diff is impossible (RNG·shuffle·time·float)
- If gacha/random/time values/float summation feed the return directly, byte diff itself does not hold.
- Priority.
  1. Seed can be fixed → raw diff before==after under the same seed (e.g. `mt_srand(고정값)`) (recommended).
  2. Not possible → switch to invariant checks — is count·key set·type·sorted value distribution·sum·probability table preserved.
  3. Prove in code that the RNG/time source itself was untouched (e.g. only cost calc was batched, the draw logic is unchanged).
- Optimizations that change the RNG/time source are forbidden (behavior change).
- In this case the candidate table "return 영향" column is not "없음" but "non-deterministic (invariant preserved, raw diff impossible)".

## 5. Multi-angle cases (as required — close each with before==after)
- Normal (several representative inputs).
- Boundary (0·1·bulk, min/max, first·last element).
- Empty/none (empty array·null·missing ID·0 results).
- Sort·keys·count (order·keys·count unchanged, sort stability).
- State variety (cache hit/miss, new/existing user, permission/condition branch).
- Repeated calls (no omission·contamination on cache/batch switch).
- For non-deterministic methods, judge each angle by "identity under fixed seed" or "distribution·count·key-set identical". Angles that cannot be compared get N/A + reason.
