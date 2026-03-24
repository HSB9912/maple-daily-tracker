# Architecture

**Analysis Date:** 2026-03-25

## Pattern Overview

**Overall:** Single-file monolithic SPA (no build step, no framework)

**Key Characteristics:**
- Everything lives in one `index.html` file (~1720 lines): HTML, CSS, and JavaScript
- No module system, bundler, or framework — pure vanilla JS with global functions
- Data flows through a single global object `_cloudData` that mirrors a multi-user JSON structure
- UI is rendered imperatively via `innerHTML` string concatenation (no virtual DOM, no templates)
- Persistence uses a dual-layer strategy: localStorage cache + GitHub Gist cloud sync

## Layers

**Presentation (HTML + CSS):**
- Purpose: Static DOM skeleton and all styling
- Location: `index.html` lines 1–287 (HTML), lines 7–175 (CSS `<style>` block)
- Contains: App shell with tab panels (재획 일지, 가계부, 정산, 설정), calendar widget, spreadsheet tables, dashboard cards, toast notification
- Depends on: Nothing (static markup)
- Used by: JavaScript render functions that populate `innerHTML` of container elements

**State Management (Global JS variables):**
- Purpose: Hold application state in memory
- Location: `index.html` lines 290–304, 511–521, 586–591, 1152
- Contains: Key globals — `_cloudData` (all user data), `_gistId`, `_unsaved`, `_saving`, `_saveQueued`, `calYear`, `calMonth`, `currentView`
- Depends on: Nothing
- Used by: All render and data functions

**Data Layer (load/save functions):**
- Purpose: Read and write per-user profile data within `_cloudData`
- Location: `index.html` lines 306–509
- Contains: `getCurrentUser()`, `setCurrentUser()`, `loadData()`, `saveData()`, `cleanEmptyRecords()`, `getEmptyCloud()`
- Depends on: `_cloudData` global, `localStorage`
- Used by: Every feature module (input, history, settings, ledger, dashboard)

**Cloud Sync Layer (Gist API):**
- Purpose: Persist data to GitHub Gist for cross-device access
- Location: `index.html` lines 291–396
- Contains: `cloudSave()`, `cloudLoad()`, `initCloud()`, `manualCloudSave()`, `manualCloudLoad()`
- Depends on: GitHub Gist REST API, hardcoded token (`_tk` char array at line 293), `FIXED_GIST_ID` at line 298
- Used by: Init flow, save button, load button

**Feature Modules (render + logic functions):**

*Input Tab (재획 일지):*
- Purpose: Daily resource tracking spreadsheet with before/after values per character
- Location: `index.html` lines 581–901 (calendar + input table + keyboard nav)
- Contains: `renderCal()`, `renderInputTable()`, `updateValue()`, `updateSums()`, keyboard navigation handler
- Depends on: Data layer, date input element

*History/Settlement Tab (정산):*
- Purpose: Accumulated settlement per character + daily/weekly/monthly aggregated views
- Location: `index.html` lines 1010–1360
- Contains: `getAccumulated()`, `settleChar()`, `rollbackSettle()`, `renderHistory()`, `renderRecordView()`, `renderDailyView()`, `renderGroupedView()`, `getWeekInfo()`
- Depends on: Data layer, settlement data structure

*Ledger Tab (가계부):*
- Purpose: Income/expense tracking in meso (억 unit)
- Location: `index.html` lines 1622–1698
- Contains: `addLedger()`, `deleteLedger()`, `renderLedger()`, `parseMeso()`, `formatMeso()`
- Depends on: Data layer, `data.ledger` array

*Settings Tab (설정):*
- Purpose: Manage characters, resources, import/export/reset
- Location: `index.html` lines 1362–1550
- Contains: `renderSettings()`, `renderList()`, `startRename()`, `addCharacter()`, `addResource()`, `deleteItem()`, `moveItem()`, `exportData()`, `importData()`, `resetData()`
- Depends on: Data layer

*Dashboard:*
- Purpose: Summary cards showing weekly meso income, total income/expense/net for current user
- Location: `index.html` lines 1552–1620
- Contains: `renderDashboard()`, `formatMeso()`
- Depends on: Data layer, `_cloudData.users` for multi-user summary

*User Management:*
- Purpose: Multi-user tab switching, add/delete/rename users
- Location: `index.html` lines 398–472
- Contains: `renderUserSelect()`, `switchUser()`, `addUser()`, `deleteUserTab()`, `renameUserTab()`
- Depends on: `_cloudData.users`, `_cloudData.profiles`

## Data Flow

**App Initialization (lines 1700–1709):**

1. Read `maple_cloud_cache` from localStorage into `_cloudData` for instant render
2. Call `renderUserSelect()`, `renderDashboard()`, `renderCal()`, `renderInputTable()` — immediate UI
3. Call `initCloud()` asynchronously which fetches from Gist API
4. If cloud data is newer/valid, replace `_cloudData` and re-render via `refreshAll()`
5. Legacy migration: if old `maple_tracker_users` or `maple_daily_tracker` keys exist in localStorage, import them

**User Input → Save (data entry in spreadsheet):**

1. User types in a cell input → `input` event on `inputTableWrap` container (delegated)
2. `updateValue(el)` reads `data-*` attributes (date, char, res, field) to locate the record
3. Calls `loadData()` → mutates the record object → calls `saveData(data)`
4. `saveData()` runs `cleanEmptyRecords()`, writes to `_cloudData.profiles[username]`, writes to localStorage, calls `markUnsaved()`
5. `updateSums()` recalculates earned values and totals in the DOM
6. User clicks "저장하기" button → `manualCloudSave()` → `cloudSave()` PATCHes the Gist

**State Management:**
- All state lives in the `_cloudData` global object (in-memory)
- localStorage key `maple_cloud_cache` mirrors `_cloudData` (written on every `saveData()` call)
- localStorage key `maple_current_user` tracks which user tab is active
- No reactive bindings — every state change requires explicit re-render calls
- The `_unsaved` flag tracks whether local changes need cloud sync

## Key Abstractions

**Cloud Data Structure (`_cloudData`):**
- Purpose: Top-level container for all users and their profiles
- Shape: `{ users: string[], profiles: { [userName]: UserProfile } }`
- UserProfile shape: `{ characters: string[], resources: string[], records: {}, settlements: {}, ledger: [] }`

**Records Structure (`data.records`):**
- Purpose: Daily resource tracking data
- Shape: `{ [dateStr]: { [charName]: { [resName]: RecordValue } } }`
- RecordValue for normal resources: `{ before: number|'', after: number|'' }`
- RecordValue for 경험치 (EXP): `{ beforeLv: number|'', beforePct: number|'', afterLv: number|'', afterPct: number|'' }`

**Settlements Structure (`data.settlements`):**
- Purpose: Track settlement history per character
- Shape: `{ [charName]: { lastDate: string, history: [{ date: string, amounts: { [resName]: number } }] } }`

**Ledger Structure (`data.ledger`):**
- Purpose: Income/expense log
- Shape: `[{ date: string, type: 'income'|'expense', amount: number, memo: string, id: number }]`
- Amount stored in raw meso (input is in 억 units, multiplied by 100,000,000)

## Entry Points

**Page Load:**
- Location: `index.html` lines 1700–1709
- Triggers: Browser opens the file
- Responsibilities: Bootstrap from localStorage cache, render initial UI, start async cloud sync

**Tab Click Handlers:**
- Location: `index.html` lines 568–579
- Triggers: User clicks tab buttons (재획 일지, 가계부, 정산, 설정)
- Responsibilities: Toggle panel visibility, call appropriate render function

**Cloud Save/Load Buttons:**
- Location: `index.html` lines 182–184 (HTML), 523–548 (JS)
- Triggers: User clicks 저장하기/불러오기 buttons in header
- Responsibilities: Push to or pull from GitHub Gist

## Error Handling

**Strategy:** Minimal — mostly try/catch around network calls, user-facing errors via toast notifications

**Patterns:**
- Network errors (Gist API): caught silently with `console.error`, no retry logic
- User input validation: guard clauses with early return + toast message (e.g., duplicate names, empty inputs)
- Data reset: requires password "0000" + confirm dialog as safety gate
- Unsaved changes: `beforeunload` event warns user before closing tab

## Cross-Cutting Concerns

**Logging:** `console.error` only for Gist failures; no structured logging
**Validation:** Inline checks in each function; input fields strip non-numeric characters via regex `el.value.replace(/[^0-9.\-]/g, '')`
**Authentication:** None for users; Gist API uses a hardcoded personal access token (embedded as charCode array at line 293)
**Internationalization:** All UI text is hardcoded Korean; no i18n framework
**Week Boundary:** Thursday-to-Wednesday week cycle (MapleStory reset schedule), implemented in `getWeekInfo()` at line 1182

---

*Architecture analysis: 2026-03-25*
