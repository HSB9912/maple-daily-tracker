# Technology Stack

**Analysis Date:** 2026-03-25

## Languages

**Primary:**
- HTML5 - Application markup, single-file architecture (`index.html`)
- CSS3 - Inline `<style>` block, all styling embedded in `index.html` (lines 7-175)
- JavaScript (ES2017+) - Inline `<script>` block, all logic embedded in `index.html` (lines 290-1718)

**Secondary:**
- None

## Runtime

**Environment:**
- Browser-only (any modern browser with ES2017+ support)
- No server-side runtime required
- No Node.js dependency

**Package Manager:**
- None - zero build dependencies
- No `package.json`, `node_modules`, or lockfile

## Frameworks

**Core:**
- None - pure vanilla HTML/CSS/JavaScript with no framework
- DOM manipulation via `document.getElementById()`, `document.querySelectorAll()`, `innerHTML` assignment
- No virtual DOM, no component system, no templating engine

**Testing:**
- None - no test framework, no test files

**Build/Dev:**
- None - no build step, no bundler, no transpiler
- The app runs directly from `index.html` opened in a browser or served via GitHub Pages

## Key Dependencies

**Critical:**
- None (zero external dependencies)
- No CDN imports, no `<script src>` tags, no CSS framework links

**Browser APIs Used:**
- `fetch()` - GitHub Gist API communication (`index.html` lines 326-333, 345-356)
- `localStorage` - Local data caching and user preference storage (`index.html` lines 307-311, 361-367, 432, 446, 507)
- `FileReader` - JSON import functionality (`index.html` lines 1523-1537)
- `Blob` / `URL.createObjectURL()` - JSON export/download (`index.html` lines 1510-1516)
- `beforeunload` event - Unsaved changes warning (`index.html` lines 1712-1717)

## Configuration

**Environment:**
- Gist API token is embedded directly in `index.html` as a charCode array (line 293)
- Gist ID is hardcoded as `FIXED_GIST_ID` constant (line 298)
- No `.env` files, no environment variable system
- No configuration files of any kind

**Build:**
- No build configuration
- No `tsconfig.json`, no `webpack.config.js`, no `vite.config.js`

## Platform Requirements

**Development:**
- Any text editor
- A web browser for testing
- Git for version control

**Production:**
- GitHub Pages static hosting (serves `index.html` from repository root)
- No server, no database, no runtime environment required
- Repository: `HSB9912/maple-daily-tracker` on GitHub

## Architecture Notes

**Single-file architecture:**
- The entire application (HTML + CSS + JS) lives in one file: `index.html` (~1720 lines, ~71KB)
- CSS: lines 7-175 (~168 lines)
- HTML body: lines 177-288 (~111 lines)
- JavaScript: lines 290-1718 (~1428 lines)

**JavaScript patterns:**
- All functions are global (no modules, no IIFE wrapping for isolation)
- Uses `async/await` for Gist API calls
- Uses template literals extensively for HTML generation
- Uses `let`/`const` (no `var`)
- Korean language throughout (variable names are English, UI strings are Korean)

**CSS patterns:**
- Dark theme (background `#121212`, text `#e0e0e0`)
- Accent color: `#f5a623` (orange/gold - MapleStory themed)
- Monospace font for numeric data: `Consolas`, `Courier New`
- UI font stack: `Segoe UI`, `Malgun Gothic`, `-apple-system`, `sans-serif`
- CSS animations: `pulse` keyframe for unsaved indicator

---

*Stack analysis: 2026-03-25*
