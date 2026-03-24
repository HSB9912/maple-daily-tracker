# Testing Patterns

**Analysis Date:** 2026-03-25

## Test Framework

**Runner:** None

**Assertion Library:** None

**Run Commands:**
```bash
# No test commands exist. There is no test suite.
```

## Current State

**There are zero automated tests in this project.** No test framework is installed, no test files exist, and no testing infrastructure is configured. The project has no `package.json`, no build system, and no dependency management.

## Test File Organization

**Location:** Not applicable -- no test files exist.

**Naming:** Not applicable.

## Why Testing Is Absent

This is a single-file HTML application (`index.html`, 1720 lines) with:
- No package manager (no `package.json`, `node_modules/`)
- No build tools (no bundler, no transpiler)
- No module system (all JS is in one `<script>` block, all functions global)
- Pure vanilla JS with no framework

All quality assurance is currently manual browser testing.

## Coverage

**Requirements:** None enforced.

**Estimated Coverage:** 0% automated, 100% manual.

## What Would Need Testing

**High-priority areas (complex logic, data integrity):**

1. **Data calculation functions** -- pure functions, easily testable:
   - `getEarned(rec)` (line 693): Calculates resource gain from before/after values
   - `getExpEarned(rec)` (line 670): Formats experience gain as level+percent string
   - `getExpEarnedNum(rec)` (line 684): Calculates numeric experience gain
   - `formatNum(n)` (line 562): Number formatting with locale
   - `formatMeso(n)` (line 1613): Meso currency display formatting (억/만 units)
   - `parseMeso(str)` (line 1624): Parses user input to meso integer
   - `todayStr()` (line 557): Date string generation
   - `getWeekInfo(dateStr)` (line 1182): Thursday-Wednesday week calculation
   - `getAccumulated(data, ch)` (line 1020): Accumulated unsettled totals per character

2. **Data mutation functions** -- modify state, need integration testing:
   - `saveData(data)` / `loadData()` (lines 502, 475): Data persistence
   - `cleanEmptyRecords(data)` (line 482): Cleans up sparse record objects
   - `updateValue(el)` (line 790): Input handler that mutates record data
   - `addLedger()` (line 1630): Adds income/expense entry
   - `settleChar(ch)` / `rollbackSettle(ch)` (lines 1038, 1055): Settlement operations

3. **Rename propagation** -- renames must update all references:
   - `renameCharInline(td)` (line 861): Character rename in records + settlements
   - `startRename(span)` (line 1385): Character/resource rename in settings
   - `renameUserTab(oldName)` (line 452): User profile rename

4. **Cloud sync** -- async operations with conflict potential:
   - `cloudSave()` / `cloudLoad()` (lines 318, 342): Gist API interactions
   - `initCloud()` (line 359): Boot sequence with migration from localStorage
   - Queue/dedup logic: `_saving` / `_saveQueued` flags (lines 319, 339)

## Recommended Testing Approach

**If tests are to be added, the recommended path is:**

1. **Extract pure functions** into a separate `logic.js` module
2. **Use Vitest or Jest** with jsdom for DOM-dependent code
3. **Start with unit tests** for calculation functions (highest value, lowest effort)

**Example test structure (hypothetical):**
```javascript
// tests/calculations.test.js
import { getEarned, getExpEarnedNum, formatMeso, parseMeso } from '../logic.js';

describe('getEarned', () => {
  test('returns difference of after - before', () => {
    expect(getEarned({ before: 100, after: 250 })).toBe(150);
  });
  test('returns 0 for empty record', () => {
    expect(getEarned(null)).toBe(0);
    expect(getEarned(undefined)).toBe(0);
  });
  test('handles legacy number format', () => {
    expect(getEarned(42)).toBe(42);
  });
  test('returns 0 when before or after is empty string', () => {
    expect(getEarned({ before: '', after: 100 })).toBe(0);
    expect(getEarned({ before: 100, after: '' })).toBe(0);
  });
});

describe('getExpEarnedNum', () => {
  test('calculates total percent from level difference', () => {
    expect(getExpEarnedNum({ beforeLv: 250, beforePct: 50.000, afterLv: 251, afterPct: 10.500 }))
      .toBeCloseTo(60.5);
  });
  test('returns 0 for incomplete records', () => {
    expect(getExpEarnedNum({ beforeLv: 250, beforePct: '' })).toBe(0);
  });
});

describe('formatMeso', () => {
  test('formats billions as 억', () => {
    expect(formatMeso(500000000)).toBe('5.00억');
  });
  test('formats ten-thousands as 만', () => {
    expect(formatMeso(50000)).toBe('5만');
  });
  test('returns 0 for zero', () => {
    expect(formatMeso(0)).toBe('0');
  });
});

describe('parseMeso', () => {
  test('converts 억 unit input to raw number', () => {
    expect(parseMeso('5.43')).toBe(543000000);
  });
  test('handles integer input', () => {
    expect(parseMeso('30')).toBe(3000000000);
  });
});
```

## Test Types

**Unit Tests:**
- Not present. Would cover pure calculation functions listed above.

**Integration Tests:**
- Not present. Would cover data flow: input -> save -> render cycle, rename propagation, settlement workflow.

**E2E Tests:**
- Not present. Would cover full user workflows: add character, enter daily data, settle, view history. Playwright or Cypress would be appropriate since this is a browser-only app.

## Risk Assessment

**Untested areas with highest risk:**

1. **`cleanEmptyRecords(data)`** (line 482) -- runs on every save, mutates data in place. A bug here silently deletes user data.
   - Files: `index.html` lines 482-499
   - Risk: High. Data loss potential.

2. **Rename propagation** -- must update records, settlements, and profiles consistently.
   - Files: `index.html` lines 861-885, 1385-1443, 452-464
   - Risk: High. Inconsistent rename = orphaned data.

3. **`getWeekInfo(dateStr)`** (line 1182) -- Thursday-to-Wednesday week math with month/year boundaries.
   - Files: `index.html` lines 1182-1204
   - Risk: Medium. Edge cases at year boundaries.

4. **`initCloud()` migration logic** (line 359) -- one-time migration from old localStorage format.
   - Files: `index.html` lines 359-396
   - Risk: Medium. Hard to manually verify all migration paths.

5. **Keyboard navigation** (lines 903-1008) -- arrow key / tab / enter navigation in spreadsheet grid.
   - Files: `index.html` lines 903-1008
   - Risk: Low-medium. Broken navigation is annoying but not data-destructive.

---

*Testing analysis: 2026-03-25*
