# Coding Conventions

**Analysis Date:** 2026-03-25

## Project Shape

**Single-file application:** Everything lives in `index.html` (1720 lines). There are no separate CSS, JS, or asset files. The structure is:
- Lines 1-6: HTML head/meta
- Lines 7-175: `<style>` block (all CSS)
- Lines 177-288: HTML body markup
- Lines 290-1718: `<script>` block (all JavaScript)

## Naming Patterns

**Functions:**
- Use camelCase: `renderCal()`, `loadData()`, `saveData()`, `getEarned()`, `formatMeso()`
- Render functions prefixed with `render`: `renderCal()`, `renderInputTable()`, `renderHistory()`, `renderSettings()`, `renderDashboard()`, `renderLedger()`, `renderList()`, `renderGroupedView()`, `renderDailyView()`, `renderRecordView()`, `renderHistoryView()`, `renderUserSelect()`
- Action functions use verb-noun: `addUser()`, `deleteItem()`, `moveItem()`, `switchUser()`, `settleChar()`, `rollbackSettle()`, `exportData()`, `importData()`, `resetData()`, `addLedger()`, `deleteLedger()`
- Getter functions prefixed with `get`: `getEarned()`, `getExpEarned()`, `getExpEarnedNum()`, `getAccumulated()`, `getCurrentUser()`, `getEmptyCloud()`, `getGistToken()`, `getWeekInfo()`
- Cloud operations prefixed with `cloud`/`manual`: `cloudSave()`, `cloudLoad()`, `initCloud()`, `manualCloudSave()`, `manualCloudLoad()`

**Variables:**
- camelCase for all variables: `calYear`, `calMonth`, `currentView`, `weekIncome`, `totalExpense`
- Short abbreviations common: `ch` (character), `r` (resource), `d` (date/day), `el` (element), `rec` (record), `sh` (settlement HTML), `bv`/`av` (before value / after value), `bLv`/`bPct` (before level / before percent)
- Module-level state prefixed with underscore: `_cloudData`, `_gistId`, `_saving`, `_saveQueued`, `_unsaved`, `_tk`, `_cachedBoot`
- Constants in UPPER_SNAKE_CASE: `GIST_API`, `GIST_FILE`, `FIXED_GIST_ID`, `DEFAULT_RESOURCES`

**CSS Classes:**
- Kebab-case throughout: `cloud-btn`, `user-tab`, `cal-day`, `dash-card`, `cell-earned`, `row-header`
- BEM-like naming without strict BEM: `cloud-btn-load`, `user-tab-x`, `user-tab-add`, `cal-today-btn`
- State classes: `.active`, `.unsaved`, `.selected`, `.today`, `.has-data`, `.other-month`, `.sun`, `.sat`
- Component prefixes group related styles: `cal-*` (calendar), `dash-*` (dashboard), `cell-*` (spreadsheet cells), `btn-*` (buttons)

**HTML IDs:**
- camelCase: `cloudSaveBtn`, `cloudLoadBtn`, `userTabs`, `inputTableWrap`, `calDays`, `calTitle`, `histStart`, `histEnd`, `ledgerDate`, `ledgerType`, `ledgerAmount`, `ledgerMemo`

**Data attributes:**
- Kebab-case: `data-tab`, `data-view`, `data-date`, `data-char`, `data-res`, `data-field`, `data-charname`, `data-earned-char`, `data-earned-res`, `data-sum-res`, `data-type`, `data-index`

## Code Style

**Formatting:**
- No formatter or linter configured (no `.eslintrc`, `.prettierrc`, or similar)
- 2-space indentation in HTML/CSS sections
- No consistent indentation in JS (mix of 2-space in functions)
- Semicolons used consistently at end of statements
- Single quotes for strings in JS, double quotes in HTML attributes
- Template literals (backticks) used extensively for HTML generation

**Line Length:**
- No enforced limit. Many lines exceed 200 characters, especially HTML-generating lines (e.g., lines 736-738 are 300+ chars)

**Braces:**
- Opening brace on same line as function/if/for
- Short statements often on single line: `if (!name) return;`
- Multi-statement blocks use braces

## Import Organization

**Not applicable.** No module system. All code is in a single `<script>` block. All functions are global.

**Path Aliases:**
- None. No build system, no modules.

## Error Handling

**Patterns:**
- `try/catch` for async network operations only (`cloudSave()`, `cloudLoad()` at lines 318-340, 342-357)
- Errors logged to `console.error()` with Korean descriptions
- User-facing errors shown via `toast(message, 'error')` function (line 550)
- Input validation uses early returns: `if (!name || !name.trim()) return;`
- Defensive null checks with optional chaining: `data.records?.[date]?.[ch]?.[r]`
- `confirm()` dialogs guard destructive operations (delete, reset, settle)
- `typeof` checks before accessing object properties: `if (typeof rec !== 'object' || rec === null)` (lines 671, 695, 801, 813)

## Logging

**Framework:** `console.error()` only, no `console.log()` or structured logging.

**Patterns:**
- Only used for cloud sync failures (lines 333, 354)
- Korean-language error messages: `'Gist 저장 실패:'`, `'Gist 로드 실패:'`
- No debug logging, no log levels

## Comments

**When to Comment:**
- Section headers as separator comments: `// --- GitHub Gist ---`, `// --- 유저 관리 ---`, `// --- Input Tab ---`, `// --- Tabs ---`
- Inline comments for non-obvious logic, written in Korean: `// 이 날짜가 이 캐릭터의 정산 범위에 포함되는지` (line 718)
- Brief functional comments: `// 빈 체크` (line 804), `// 레코드에서 이름도 변경` (line 1410)
- CSS section comments: `/* Calendar */`, `/* Spreadsheet */`, `/* Settled */`, `/* Dashboard */`, `/* Ledger */`

**JSDoc/TSDoc:**
- Not used. No type annotations anywhere.

## Function Design

**Size:**
- Most functions 5-30 lines
- `renderInputTable()` is the largest at ~85 lines (703-788)
- `renderCal()` is ~50 lines (615-667)
- Keyboard handler is ~100 lines (903-1008) as a single event listener

**Parameters:**
- Positional parameters, no options objects
- Functions typically accept 0-3 parameters
- DOM elements passed directly: `updateValue(el)`, `startRename(span)`, `renameCharInline(td)`

**Return Values:**
- Data functions return plain objects or primitives
- Void functions for rendering and side effects (majority of codebase)
- `getEarned()` returns number, defaults to 0
- `getExpEarned()` returns formatted string or empty string

## Module Design

**Exports:** Not applicable (no modules)

**Barrel Files:** Not applicable

**Code Organization within `<script>` block:**
1. Constants and Gist config (lines 291-304)
2. User management functions (lines 306-472)
3. Data load/save/clean (lines 474-521)
4. Cloud sync UI functions (lines 523-565)
5. Tab switching (lines 567-579)
6. Calendar logic (lines 581-667)
7. Data calculation helpers (lines 669-701)
8. Input table rendering (lines 703-788)
9. Value update and keyboard navigation (lines 790-1008)
10. History/settlement rendering (lines 1010-1360)
11. Settings management (lines 1362-1505)
12. Export/Import/Reset (lines 1507-1550)
13. Dashboard (lines 1552-1620)
14. Ledger/accounting (lines 1622-1698)
15. Init/boot sequence (lines 1700-1718)

## HTML Generation Pattern

**Primary pattern:** Build HTML strings with concatenation and template literals, then assign to `element.innerHTML`.

```javascript
// Typical pattern (from renderInputTable, line 713):
let html = '<table class="sheet"><thead><tr><th class="corner"></th>';
data.resources.forEach(r => html += `<th>${r}</th>`);
html += '</tr></thead><tbody>';
// ... build rows ...
wrap.innerHTML = html;
```

- All render functions follow this pattern
- Event handlers attached via inline `onclick` attributes in generated HTML: `onclick="settleChar('${ch}')"`
- Some event delegation used for the input spreadsheet (lines 890-1008)

## Data Model Convention

**Shape:** All data stored as plain JSON objects. No classes or prototypes.

```javascript
// Per-user profile structure:
{
  characters: string[],      // ordered list of character names
  resources: string[],       // ordered list of resource names
  records: {                 // keyed by date string "YYYY-MM-DD"
    [date]: {
      [charName]: {
        [resName]: { before: number|'', after: number|'' }
        // or for 경험치:
        [resName]: { beforeLv: number|'', beforePct: number|'', afterLv: number|'', afterPct: number|'' }
      }
    }
  },
  settlements: {
    [charName]: {
      lastDate: string,      // "YYYY-MM-DD"
      history: Array<{ date: string, amounts: { [resName]: number } }>
    }
  },
  ledger: Array<{ date: string, type: 'income'|'expense', amount: number, memo: string, id: number }>
}
```

## State Management

- Global mutable variables for app state: `_cloudData`, `_unsaved`, `calYear`, `calMonth`, `currentView`
- `localStorage` used as cache layer (`maple_cloud_cache`, `maple_current_user`)
- GitHub Gist used as persistent cloud storage
- No state management library, no pub/sub, no reactive binding
- State changes trigger manual re-renders via `refreshAll()` or specific `render*()` calls

## Language

- UI text is entirely in Korean
- Code comments mix Korean and English
- Variable/function names are English
- Some domain terms kept in Korean as data values: `'매소'`, `'경험치'`, `'기본'`

---

*Convention analysis: 2026-03-25*
