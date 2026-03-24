# Codebase Structure

**Analysis Date:** 2026-03-25

## Directory Layout

```
.maple/
├── .claude/                # Claude AI configuration
│   ├── settings.json       # Shared Claude settings
│   └── settings.local.json # Local Claude settings
├── .git/                   # Git repository
├── .planning/              # Planning documents (generated)
│   └── codebase/           # Codebase analysis docs
├── index.html              # THE ENTIRE APPLICATION (single file, ~1720 lines)
└── README.md               # Brief project description
```

## Single-File Architecture

This project is a **single-file application**. All code lives in `index.html`. There are no separate JS, CSS, or asset files. There is no build step, no package.json, no dependencies to install.

**`index.html` Internal Layout:**

```
Lines 1–6       HTML head, meta tags
Lines 7–175     <style> block — ALL CSS
Lines 177–286   <body> HTML structure — app shell
Lines 287–288   Toast notification element
Lines 290–1718  <script> block — ALL JavaScript
Lines 1719–1720 Closing tags
```

## Key File Locations

**Entry Point:**
- `index.html`: The only source file. Open in a browser to run.

**Configuration:**
- `index.html` line 292: `GIST_API` constant (GitHub Gist endpoint)
- `index.html` line 293: `_tk` — Gist auth token as charCode array
- `index.html` line 297: `GIST_FILE` — filename within Gist (`maple-tracker-data.json`)
- `index.html` line 298: `FIXED_GIST_ID` — hardcoded Gist ID for cloud sync
- `index.html` line 304: `DEFAULT_RESOURCES` — default resource item names

**Core Logic (within `<script>` block of `index.html`):**

| Line Range | Section | Key Functions |
|------------|---------|---------------|
| 290–304 | Constants & globals | `GIST_API`, `_tk`, `_cloudData`, `DEFAULT_RESOURCES` |
| 306–311 | User state | `getCurrentUser()`, `setCurrentUser()` |
| 313–396 | Cloud sync | `cloudSave()`, `cloudLoad()`, `initCloud()` |
| 398–472 | User management | `addUser()`, `deleteUserTab()`, `renameUserTab()`, `switchUser()` |
| 474–509 | Data load/save | `loadData()`, `saveData()`, `cleanEmptyRecords()` |
| 511–565 | UI helpers | `markUnsaved()`, `markSaved()`, `manualCloudSave()`, `toast()`, `todayStr()`, `formatNum()` |
| 567–579 | Tab switching | Event listeners on `.tab` elements |
| 581–667 | Calendar | `calMove()`, `calToday()`, `calSelect()`, `renderCal()` |
| 669–701 | EXP calculations | `getExpEarned()`, `getExpEarnedNum()`, `getEarned()` |
| 703–788 | Input table render | `renderInputTable()` |
| 790–858 | Cell update logic | `updateValue()`, `updateSums()` |
| 860–885 | Char rename inline | `renameCharInline()` |
| 887–1008 | Keyboard navigation | Delegated `input`, `focusin`, `keydown` handlers on `inputTableWrap` |
| 1010–1071 | Settlement logic | `getAccumulated()`, `settleChar()`, `rollbackSettle()` |
| 1073–1360 | History/settlement render | `renderHistory()`, `renderRecordView()`, `renderDailyView()`, `renderGroupedView()`, `getWeekInfo()` |
| 1362–1501 | Settings tab | `renderSettings()`, `renderList()`, `startRename()`, `addCharacter()`, `addResource()`, `deleteItem()`, `moveItem()` |
| 1507–1550 | Export/Import/Reset | `exportData()`, `importData()`, `resetData()` |
| 1552–1620 | Dashboard | `renderDashboard()`, `formatMeso()` |
| 1622–1698 | Ledger (가계부) | `parseMeso()`, `addLedger()`, `deleteLedger()`, `renderLedger()` |
| 1700–1709 | Init/Bootstrap | Cache load → render → async cloud sync |
| 1711–1717 | Beforeunload guard | Warns on unsaved changes |

## HTML Structure (DOM IDs)

**Top-level containers:**
- `#userTabs` — User tab bar
- `#dashboard` — Summary dashboard cards
- `#input` — Daily input panel (active by default)
- `#ledger` — Income/expense panel
- `#history` — Settlement + history panel
- `#settings` — Settings panel
- `#toast` — Toast notification overlay

**Key interactive elements:**
- `#cloudSaveBtn`, `#cloudLoadBtn` — Cloud sync buttons
- `#inputDate` — Hidden date input (synced with calendar)
- `#calTitle`, `#calDays` — Calendar header and grid
- `#inputTableWrap` — Input spreadsheet container
- `#ledgerDate`, `#ledgerType`, `#ledgerAmount`, `#ledgerMemo` — Ledger form inputs
- `#ledgerTableWrap` — Ledger table container
- `#settlementWrap` — Settlement summary table
- `#historyTableWrap` — History records table
- `#histStart`, `#histEnd` — Date range for daily view
- `#yearSelect` — Year selector for weekly/monthly views
- `#charList`, `#resList` — Character and resource lists in settings
- `#newCharName`, `#newResName` — Add-new input fields
- `#importFile` — Hidden file input for JSON import

## Naming Conventions

**Files:**
- Only one file: `index.html` (lowercase, standard HTML convention)

**Functions:**
- camelCase: `renderInputTable()`, `getAccumulated()`, `settleChar()`
- Render functions prefixed with `render`: `renderCal()`, `renderHistory()`, `renderDashboard()`, `renderLedger()`, `renderSettings()`
- Get/load functions prefixed accordingly: `loadData()`, `getCurrentUser()`, `getEarned()`
- Action functions use verb: `addUser()`, `deleteItem()`, `moveItem()`, `switchView()`

**Variables:**
- camelCase for locals: `weekIncome`, `totalExpense`, `calYear`
- Underscore prefix for module-level state: `_cloudData`, `_gistId`, `_unsaved`, `_saving`, `_saveQueued`, `_tk`

**CSS Classes:**
- kebab-case: `.cloud-btn`, `.user-tab`, `.cal-day`, `.dash-card`
- BEM-like but informal: `.cloud-btn-load`, `.cal-today-btn`, `.user-tab-x`
- State classes: `.active`, `.unsaved`, `.settled-row`, `.has-data`, `.today`, `.selected`

**Data Attributes:**
- `data-tab` on tab buttons for panel switching
- `data-view` on view toggle buttons (daily/weekly/monthly)
- `data-date`, `data-char`, `data-res`, `data-field` on cell inputs for event delegation
- `data-earned-char`, `data-earned-res` on earned-value display cells
- `data-sum-res` on sum row cells
- `data-type`, `data-index` on settings list items
- `data-charname` on row headers for inline rename

## Where to Add New Code

**New Feature Tab:**
1. Add a `<button class="tab" data-tab="newtab">` in the `.tabs` div (around line 192)
2. Add a `<div id="newtab" class="panel">` for the panel content (around line 285)
3. Add render function `renderNewTab()` in the `<script>` block
4. Add the render call in the tab click handler (line 574) and in `refreshAll()` (line 466)

**New Data Field on User Profile:**
1. Update `getEmptyCloud()` at line 313 to include the new field
2. Update `resetData()` at line 1544 to include the new field in the empty object
3. Access via `loadData()` return value; save via `saveData(data)`

**New Resource Calculation Type:**
1. Follow the pattern of 경험치 (EXP) special handling
2. Add special-case branches in `renderInputTable()` (line 728), `updateValue()` (line 799), `getEarned()` (line 693), and sum calculations

**New Dashboard Card:**
1. Add HTML string in `renderDashboard()` at line 1553
2. Follow existing pattern: `<div class="dash-card">...</div>`

**New Utility Function:**
1. Add near existing utilities (lines 557–565)
2. All functions are global scope — no imports needed

## Special Directories

**`.planning/`:**
- Purpose: Contains codebase analysis and planning documents
- Generated: Yes (by tooling)
- Committed: No (not yet tracked)

**`.claude/`:**
- Purpose: Claude AI tool configuration
- Generated: Yes
- Committed: Not yet (untracked per git status)

## Data Storage Locations

**localStorage Keys:**
- `maple_cloud_cache` — Full `_cloudData` JSON (primary local cache)
- `maple_current_user` — Active user name string
- `maple_tracker_users` — Legacy key (migrated on first run)
- `maple_daily_tracker` — Legacy key (migrated on first run)

**Remote:**
- GitHub Gist ID `ed1c7a037614886df9cf744decb9079a`, file `maple-tracker-data.json`

---

*Structure analysis: 2026-03-25*
