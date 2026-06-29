# 스택: Frontend (브라우저 / 렌더)

## 측정
- 쿼리·IO 계획에 해당: Network 워터폴(요청 수·폭포·중복 요청).
- 타이밍: Performance API(`performance.now()`, `PerformanceObserver`), React Profiler, DevTools Performance 탭.
- 정적: 불필요 리렌더(메모 누락), 리스트 `key` 누락/불안정, 루프 안 동기 레이아웃 읽기(reflow).

## 안티패턴
- 요청 N+1: 컴포넌트마다 개별 fetch → 상위에서 배치/병렬 + 캐시(React Query 등).
- 리렌더 폭증: `useMemo`/`useCallback`/`React.memo`, 안정적 key, 셀렉터 분리.
- 레이아웃 스래싱: 읽기/쓰기 분리, `requestAnimationFrame` 배치.

## return(렌더 출력) 깨짐 함정
- 메모이제이션이 stale 값을 잡아 출력이 달라짐 → 의존성 배열 정확히.
- key 변경으로 컴포넌트 재마운트 → 상태 초기화로 동작 변화.
- 정렬/필터 결과 순서 변경.

## 검증
- 렌더 출력 스냅샷(DOM/Testing Library) before/after 비교.
- Network 요청 수 N→1 확정, Performance 측정으로 시간 비교.
