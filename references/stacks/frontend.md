# Stack: Frontend (browser / render)

## Measure
- Equivalent of query/IO plan: Network waterfall (request count·cascade·duplicate requests).
- Timing: Performance API (`performance.now()`, `PerformanceObserver`), React Profiler, DevTools Performance tab.
- Static: unneeded re-render (missing memo), missing/unstable list `key`, in-loop synchronous layout read (reflow).

## Anti-patterns
- Request N+1: per-component individual fetch → batch/parallel from the parent + cache (React Query etc.).
- Re-render explosion: `useMemo`/`useCallback`/`React.memo`, stable key, selector split.
- Layout thrashing: separate read/write, batch via `requestAnimationFrame`.

## Return (render output) break traps
- Memoization captures a stale value so output differs → get the dependency array exactly right.
- key change remounts a component → state reset changes behavior.
- Sort/filter result order changed.

## Verify
- Render output snapshot (DOM/Testing Library) compare before/after.
- Confirm Network request count N→1, compare time via Performance measurement.
