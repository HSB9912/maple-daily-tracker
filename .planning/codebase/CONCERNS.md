# Codebase Concerns

**Analysis Date:** 2026-03-25

## Tech Debt

**Monolithic Single-File Architecture:**
- Issue: The entire application (HTML, CSS, JS) lives in a single 1720-line `index.html` file (~70KB). All styles are inline in a `<style>` block (lines 7-175), all markup is in one `<body>` (lines 177-288), and all logic is in a single `<script>` block (lines 290-1718). There is no module system, no build step, no separation of concerns.
- Files: `index.html`
- Impact: Extremely difficult to maintain, test, or refactor. Any change requires navigating a massive file. No code reuse. No tree-shaking. No minification.
- Fix approach: Extract into modules: `styles.css`, `app.js` (or multiple JS modules). Consider a lightweight bundler (Vite) if module splitting is desired. Alternatively, keep single-file but add clear section markers and reduce repetition in render functions.

**Global State with No Encapsulation:**
- Issue: All application state is stored in global variables (`_cloudData`, `_gistId`, `_saving`, `_saveQueued`, `_unsaved`, `calYear`, `calMonth`, `currentView`). All functions are global. No namespacing, no classes, no module pattern.
- Files: `index.html` lines 291-303, 511, 586-591, 1152
- Impact: Name collisions risk with any browser extension or embedded context. Difficult to reason about state flow. Any function can mutate `_cloudData` at any time.
- Fix approach: Wrap all code in an IIFE or module. Centralize state in a single state object with controlled access.

**Repetitive HTML String Construction:**
- Issue: Multiple render functions (`renderInputTable`, `renderHistory`, `renderCal`, `renderLedger`, `renderDashboard`, `renderGroupedView`, `renderDailyView`, `renderRecordView`) build HTML via string concatenation and assign to `innerHTML`. This pattern appears 21 times across the codebase.
- Files: `index.html` lines 703-788, 1073-1150, 1217-1347, 1553-1611, 1659-1694
- Impact: Hard to maintain, error-prone (unclosed tags), XSS-vulnerable (user input interpolated into HTML), poor performance (full DOM rebuild on every edit).
- Fix approach: Use `document.createElement` for dynamic content, or adopt a lightweight template library. At minimum, escape all user-provided strings before inserting into HTML.

**Hardcoded Reset Password:**
- Issue: The data reset function uses a hardcoded plaintext password `'0000'` checked client-side.
- Files: `index.html` line 1542
- Impact: Provides zero actual security. Anyone can read the source and see the password. The `confirm()` dialog is the only real protection.
- Fix approach: Remove the password check entirely and rely solely on the confirmation dialog, or add a more meaningful protection (e.g., type the user name to confirm).

## Security Considerations

**GitHub Personal Access Token Embedded in Source Code:**
- Risk: A GitHub PAT is embedded as a charCode array at line 293 (`const _tk = [103,104,112,...]`). The `getGistToken()` function (line 294-296) reconstructs it at runtime. This token is trivially extractable by anyone viewing the page source, browser DevTools, or the Git repository. The obfuscation (charCode array instead of plain string) provides zero security -- it only avoids automated secret scanners.
- Files: `index.html` lines 293-296
- Current mitigation: CharCode encoding (cosmetic only).
- Recommendations: **HIGH PRIORITY.** Move the token server-side. Options: (1) Use a backend proxy (Cloudflare Worker, Netlify Function) that holds the token and proxies Gist API calls. (2) Use GitHub OAuth flow so each user authenticates with their own token. (3) At minimum, scope the token to only gist access and make it a fine-grained token limited to the specific gist.

**Fixed Gist ID Shared Across All Users:**
- Risk: All users of this app read and write to the same Gist (`ed1c7a037614886df9cf744decb9079a`, line 298). There is no authentication or authorization beyond the embedded token. Any user can overwrite any other user's data.
- Files: `index.html` line 298
- Current mitigation: None. The multi-user "tabs" (lines 398-464) are purely cosmetic -- all users share the same Gist file.
- Recommendations: Each user should have their own Gist or use a backend with proper auth. If keeping shared Gist, add conflict detection (compare before/after on save).

**XSS via innerHTML with User Input:**
- Risk: User-provided strings (character names, resource names, user names, memos) are interpolated directly into HTML strings without escaping, then assigned via `innerHTML`. For example, line 722: `data-charname="${ch}"` and line 410: `onclick="switchUser('${name}')"`. A character named `<img src=x onerror=alert(1)>` or `'); alert('xss` would execute arbitrary JavaScript.
- Files: `index.html` lines 408-416 (user tabs), 714-722 (input table), 1110-1118 (settlement), 1376-1381 (settings list), 1682-1690 (ledger)
- Current mitigation: None.
- Recommendations: Escape all user input before HTML interpolation. Create a helper: `function esc(s) { const d = document.createElement('div'); d.textContent = s; return d.innerHTML; }` and use it everywhere user strings appear in HTML.

**No Import Validation:**
- Risk: The JSON import function (line 1520-1538) only checks for the presence of `characters`, `resources`, and `records` keys. It does not validate types, structure depth, or sanitize values. Malicious JSON could inject script payloads into character/resource names that execute via innerHTML.
- Files: `index.html` lines 1520-1538
- Current mitigation: Basic key existence check only.
- Recommendations: Validate imported data schema deeply. Ensure all string values are sanitized. Validate that numeric fields contain numbers.

## Performance Bottlenecks

**Full DOM Rebuild on Every Input:**
- Problem: Every keystroke in an input field triggers `updateValue()` (line 790) which calls `saveData()` (writes full `_cloudData` to localStorage as JSON), then `updateSums()` re-queries all DOM elements. The `renderInputTable()` function rebuilds the entire table HTML from scratch whenever the view changes.
- Files: `index.html` lines 890-897 (input handler), 502-509 (saveData), 825-858 (updateSums)
- Cause: `JSON.stringify(_cloudData)` on every keystroke writes the entire dataset to localStorage. For large datasets (many characters, many dates), this becomes expensive.
- Improvement path: Debounce the localStorage write (e.g., 500ms). Only update changed DOM elements in `updateSums()` rather than querying all. Consider `requestAnimationFrame` for batching.

**O(n) Record Scanning for Accumulation:**
- Problem: `getAccumulated()` (line 1020-1036) iterates over ALL record dates for each character to compute unsettled totals. `renderHistory()` calls this for every character. `renderCal()` checks every resource for every character for every day of the month.
- Files: `index.html` lines 1020-1036, 645-649, 1206-1215
- Cause: No index or precomputed summaries. All data is scanned from the flat `records` object.
- Improvement path: Maintain a running totals cache that updates incrementally on data change, rather than recomputing from scratch.

**Entire Cloud Data Serialized on Every Save:**
- Problem: `cloudSave()` serializes and sends the ENTIRE `_cloudData` object (all users, all profiles, all records) to the Gist API on every save. As data grows, this payload becomes large.
- Files: `index.html` lines 318-340
- Cause: Gist API requires full file content on PATCH; no partial update available.
- Improvement path: Split data across multiple Gist files (one per user). Or compress the JSON payload (e.g., LZ-string). Or migrate to a database backend.

## Fragile Areas

**Settlement/Rollback Data Integrity:**
- Files: `index.html` lines 1038-1071
- Why fragile: Settlement stores only the `lastDate` and a history array. Rollback simply pops the last history entry and reverts `lastDate` to the previous entry's date. If the history array is corrupted (e.g., by a failed save or concurrent edit), rollback produces incorrect state. There is no validation that the dates in settlement history are consistent with actual record dates.
- Safe modification: Always validate settlement history integrity before operating on it. Add a checksum or version counter.
- Test coverage: None (no tests exist).

**Race Condition on Cloud Save:**
- Files: `index.html` lines 318-340
- Why fragile: The save queue mechanism (`_saving` / `_saveQueued` flags) handles one queued save, but if multiple rapid saves occur, intermediate states are lost. The recursive call at line 339 (`if (_saveQueued) { _saveQueued = false; cloudSave(); }`) means only the last queued state is saved. If the Gist API call fails (line 335-337), the error is only logged to console and the data diverges silently between localStorage and cloud.
- Safe modification: Add retry logic with exponential backoff. Show persistent error state in UI (not just console.log). Implement conflict detection by checking Gist version before writing.
- Test coverage: None.

**User Rename Across Records:**
- Files: `index.html` lines 452-464 (user rename), 861-885 (character rename), 1411-1431 (settings rename)
- Why fragile: Renaming a user/character requires updating keys in multiple nested objects (`records`, `settlements`, `profiles`). If the operation is interrupted (browser crash, tab close), data can be left in a partially renamed state with orphaned records under the old name and incomplete records under the new name.
- Safe modification: Perform rename as an atomic operation: deep clone, modify clone, then assign. Or add a migration/repair function.
- Test coverage: None.

**localStorage Cache vs Cloud Desync:**
- Files: `index.html` lines 359-396 (initCloud), 700-1710 (boot sequence)
- Why fragile: On boot, the app immediately renders from localStorage cache (lines 1702-1707), then asynchronously loads from cloud (line 1709). If cloud data differs, the UI silently updates. But if the user makes edits during the async load, those edits are based on stale cache and may be overwritten when `initCloud()` completes.
- Safe modification: Disable input during initial cloud sync, or merge changes instead of overwriting.
- Test coverage: None.

## Scaling Limits

**GitHub Gist API Rate Limits:**
- Current capacity: GitHub API allows 5,000 requests/hour for authenticated users.
- Limit: With multiple users sharing one token, frequent saves could approach this limit. Each save is one PATCH request; each load is one GET request.
- Scaling path: Implement save debouncing/batching (save at most once per 30 seconds). Add rate limit header checking. Consider alternative storage (Supabase, Firebase, or IndexedDB for offline-first).

**Single Gist File Size:**
- Current capacity: GitHub Gist files can be up to 10MB.
- Limit: All user data is stored in a single JSON file. With many users, characters, and months of daily records, this file grows unbounded.
- Scaling path: Split into multiple files per user. Archive old settled records. Compress data.

**localStorage 5MB Limit:**
- Current capacity: Most browsers allow 5MB per origin in localStorage.
- Limit: The full `_cloudData` object is cached in localStorage (line 507). Same growth concern as the Gist file.
- Scaling path: Use IndexedDB for larger local storage. Or only cache the current user's data locally.

## Dependencies at Risk

**GitHub Gist as Primary Database:**
- Risk: The app uses GitHub Gist API as its sole persistent storage. GitHub could change API terms, rate limits, or deprecate the Gist API. The embedded token could be revoked/expire.
- Impact: Complete data loss if cloud save fails and localStorage is cleared.
- Migration plan: Abstract storage behind an interface (`save(data)`, `load()`) so backends can be swapped. Consider Supabase or Firebase for a proper database.

## Missing Critical Features

**No Automated Tests:**
- Problem: Zero test files exist. No test framework configured. No CI/CD pipeline.
- Blocks: Cannot safely refactor the monolithic file. Cannot verify settlement logic correctness. Cannot catch regressions.

**No Offline Support:**
- Problem: If the Gist API is unreachable, saves silently fail (errors logged to console only). The user sees no indication that their data is not persisted to cloud. The localStorage cache provides temporary resilience, but if the user clears browser data, everything is lost.
- Blocks: Reliable use on unstable connections.

**No Conflict Resolution:**
- Problem: If two browser tabs (or two devices) edit simultaneously, the last save wins. There is no optimistic locking, versioning, or merge strategy.
- Blocks: Safe multi-device usage.

**No Data Backup/Versioning:**
- Problem: Settlement history records amounts but not the underlying records. Once settled, the detail is effectively frozen but not archived separately. There is no undo beyond the single-level rollback.
- Blocks: Audit trail, debugging data discrepancies.

## Test Coverage Gaps

**All Application Logic:**
- What's not tested: Every function in the application -- data manipulation, settlement calculations, earned amount computations, rename cascading, calendar rendering, import/export, cloud sync.
- Files: `index.html` (entire `<script>` block, lines 290-1718)
- Risk: Any refactoring or bug fix could introduce regressions undetected. The settlement calculation (`getAccumulated`, `getEarned`, `getExpEarnedNum`) and date logic (`getWeekInfo`, `todayStr`) are particularly risky areas that deal with edge cases (year boundaries, timezone issues, floating point arithmetic).
- Priority: High. Extract pure functions (`getEarned`, `getExpEarnedNum`, `getAccumulated`, `getWeekInfo`, `parseMeso`, `formatMeso`, `formatNum`, `cleanEmptyRecords`) into a testable module and add unit tests first.

---

*Concerns audit: 2026-03-25*
