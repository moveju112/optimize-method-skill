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
- Call again with the same input set and reset fixtures, state, cache, and controlled external values; dump `<method>__<caseId>.after.json`.
- Use the same serializer and runtime settings on both sides. Re-encoding, sorting keys, rounding floats, or deleting time/random fields cannot prove the original bytes identical.

## 3. Compare (definition of identity)
- Default gate = is the serialized result byte-identical. This is the truth because it is what the client receives.
- Compare the original serialized bytes first. Normalized values may explain a difference but cannot turn it into PASS.
- If they differ, classify the trap with the table below (all count as "fail", do not let them pass).

| Trap | Symptom | Verdict |
|------|---------|---------|
| Assoc-array key order | JSON object key order changed | fail (preserve order) |
| int/string mix | `5` vs `"5"` | fail |
| null vs missing key | `"k":null` vs no key | fail |
| IN/batch order | result differs from input order | fail (re-sort in input order) |
| float precision | summation order changed, last digit differs | fail (deterministic arithmetic changed) |

- State the comparison level: string comparison after serialization is the default. PHP array `===` comparison is sensitive to key order/type, so use it only as an aux.

## 4. Non-deterministic output — evidence levels (RNG·shuffle·time)
- Floating-point arithmetic alone is not non-determinism. A changed operation order or tolerance is not an exception to byte identity.
- First reproduce the original output in an isolated harness: reset the seed, replay the same clock/external responses, and restore the same initial state for each before/after pair. Repeat over representative seeds, times, and states.
- Control test dependencies without changing the production RNG/time source. Check call count/order and state effects as well as returned bytes; an unchanged RNG function with a changed call sequence can change behavior.
- Controlled original bytes identical for all required cases → PASS, scoped to those cases.
- If a baseline exists but control is impossible, compare all applicable invariants before/after (count, keys, types, values/distribution, sum, probability table) and inspect source changes. Completed invariant checks without violations yield INVARIANTS_ONLY. Equal summaries or a finite sample do not prove raw identity or equal distributions.
- Any observed contract violation → FAIL. Missing baseline, incomplete invariant checks when raw control is impossible, or an unexecuted required case → UNPROVEN. Missing raw control alone does not override completed INVARIANTS_ONLY evidence.
- Never relabel INVARIANTS_ONLY/UNPROVEN as "없음", "동일", or PASS. Invariants guide the next experiment; they do not authorize retaining an unverified change.

## 5. Multi-angle cases (as required — close each with before==after)
- Normal (several representative inputs).
- Boundary (0·1·bulk, min/max, first·last element).
- Empty/none (empty array·null·missing ID·0 results).
- Sort·keys·count (order·keys·count unchanged, sort stability).
- State variety (cache hit/miss, new/existing user, permission/condition branch).
- Repeated calls (no omission·contamination on cache/batch switch).
- Mark each angle PASS/FAIL/INVARIANTS_ONLY/UNPROVEN. Use N/A + reason only when the angle genuinely does not apply; unavailable evidence for an applicable angle is UNPROVEN.
