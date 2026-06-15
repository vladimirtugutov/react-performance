# Performance Report

## Baseline (Before Optimization)

| Interaction | Render Duration | Notes |
|---|---|---|
| Sorting | 114.7ms | CountryList 93.6ms, all cards re-rendered |
| Year change | 27.7ms | YearSelector + CountryList re-rendered |
| Search | 23.5ms | Full list re-rendered on every keystroke |
| Toggle columns | 23ms | YearSelector re-rendered unnecessarily |

### Observations
- Every interaction caused full re-render of all CountryCard components
- `filteredCountries` was recalculated on every render
- `createYearDataMap` was called for every country on every sort
- All DOM nodes for all countries were mounted at once (no virtualization)
- `key={index}` caused unnecessary reconciliation

---

## Optimizations Applied

### 1. `useCallback` for all event handlers in `App`
Prevents handler recreation on every render, allows memoized child components to skip re-renders.

### 2. `useMemo` for `years` and `availableColumns` in `App`
Avoids recomputing static data on every render.

### 3. `useMemo` for `filteredCountries` in `CountryList`
Filter + sort recalculates only when dependencies change.

### 4. `React.memo` on `CountryCard`, `DataTable`, `SearchBar`, `YearSelector`, `ColumnModal`
Components skip re-render when their props haven't changed.

### 5. `useMemo` for `yearDataMap` in `CountryCard`
Map is not recreated on every render.

### 6. Fixed `key` props
`key={index}` → `key={country.id}` and `key={column}` for correct reconciliation.

### 7. Virtualization with `react-virtuoso`
Only visible country cards are mounted in the DOM instead of all ~300.

---

## Final Results (After Optimization)

| Interaction | Render Duration | Improvement |
|---|---|---|
| Sorting | 23.3ms | **-80%** (was 114.7ms) |
| Year change | 42.9ms | **-0%** (YearSelector heavy first render) |
| Search | 4.6ms | **-80%** (was 23.5ms) |
| Toggle columns | 14ms | **-39%** (was 23ms) |

### Observations
- `YearSelector` is now striped (grey) in flame chart = skipped re-render in most cases
- `CountryList` renders only visible items via virtualization
- `CountryCard` components are memoized and skip re-renders when props unchanged
- Search is now the fastest interaction at 4.6ms
