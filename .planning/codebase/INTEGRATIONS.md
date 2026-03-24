# External Integrations

**Analysis Date:** 2026-03-25

## APIs & External Services

**GitHub Gist API (Primary Cloud Storage):**
- Purpose: Persistent cloud storage for all user data across devices
- API Base: `https://api.github.com/gists` (constant `GIST_API`, `index.html` line 292)
- SDK/Client: Native `fetch()` API, no SDK
- Auth: Personal Access Token embedded as charCode array in `_tk` variable (`index.html` line 293)
- Token retrieval: `getGistToken()` function decodes charCode array to string (`index.html` lines 294-296)
- Fixed Gist ID: `ed1c7a037614886df9cf744decb9079a` (`index.html` line 298)
- Gist filename: `maple-tracker-data.json` (`index.html` line 297)

**API Operations:**

1. **Save (PATCH):**
   - Endpoint: `PATCH /gists/{gist_id}`
   - Headers: `Authorization: token {token}`, `Content-Type: application/json`
   - Body: `{ files: { "maple-tracker-data.json": { content: JSON.stringify(data) } } }`
   - Implementation: `cloudSave()` function (`index.html` lines 318-340)
   - Queue mechanism: If save is in progress, queues next save (`_saving`, `_saveQueued` flags)

2. **Load (GET):**
   - Endpoint: `GET /gists/{gist_id}`
   - Headers: `Authorization: token {token}`
   - Response parsing: `json.files["maple-tracker-data.json"].content` parsed as JSON
   - Implementation: `cloudLoad()` function (`index.html` lines 342-357)

**Save Trigger:**
- Manual only - user clicks "저장하기" button
- No auto-save to Gist (auto-save only to localStorage)
- Unsaved indicator: button pulses orange when local changes exist

## Data Storage

**Primary (Cloud):**
- GitHub Gist - single JSON file containing all users and profiles
- Connection: Hardcoded Gist ID + embedded token
- Client: Native `fetch()`

**Secondary (Local Cache):**
- `localStorage` key: `maple_cloud_cache` - full mirror of cloud data (`index.html` lines 323, 367, 432, 446, 507)
- `localStorage` key: `maple_current_user` - active user tab name (`index.html` lines 307-311)
- Purpose: Instant boot (renders from cache before cloud sync completes, `index.html` lines 1701-1709)

**Legacy localStorage keys (migration support):**
- `maple_tracker_users` - old multi-user format (`index.html` line 373)
- `maple_tracker_{username}` - old per-user data (`index.html` line 380)
- `maple_daily_tracker` - original single-user format (`index.html` line 374)
- Migration runs once during `initCloud()` if no cloud data exists (`index.html` lines 371-388)

**File Storage:**
- JSON export/import via browser download/upload (no server storage)
- Export: `exportData()` creates downloadable `.json` file (`index.html` lines 1508-1518)
- Import: `importData()` reads uploaded `.json` via FileReader (`index.html` lines 1520-1538)

**Caching:**
- localStorage serves as offline cache
- Boot sequence: render from cache immediately, then sync from Gist in background (`index.html` lines 1701-1709)

## Data Schema

**Cloud data structure (`_cloudData`):**
```json
{
  "users": ["기본", "유저2"],
  "profiles": {
    "유저이름": {
      "characters": ["캐릭이름1", "캐릭이름2"],
      "resources": ["매소", "조각", "경험치", ...],
      "records": {
        "2026-03-25": {
          "캐릭이름": {
            "매소": { "before": 1000, "after": 2000 },
            "경험치": { "beforeLv": 270, "beforePct": 45.123, "afterLv": 270, "afterPct": 46.500 }
          }
        }
      },
      "settlements": {
        "캐릭이름": {
          "lastDate": "2026-03-20",
          "history": [{ "date": "2026-03-20", "amounts": { "매소": 50000 } }]
        }
      },
      "ledger": [
        { "date": "2026-03-25", "type": "income", "amount": 543000000, "memo": "보스 드랍", "id": 1711234567890 }
      ]
    }
  }
}
```

## Authentication & Identity

**Auth Provider:**
- None for end users - app has no login system
- GitHub token is used for API auth only (Gist read/write)
- Token is shared across all users of the app (single Gist, single token)
- No user authentication, no session management

**Security note:**
- The GitHub PAT is embedded in client-side code as a charCode array (`index.html` line 293)
- Anyone who views page source can extract the token
- The reset function uses a hardcoded password `"0000"` (`index.html` line 1542)

## Monitoring & Observability

**Error Tracking:**
- None - errors logged to `console.error()` only (`index.html` lines 333, 336, 354)

**Logs:**
- `console.error()` for Gist save/load failures
- No structured logging, no log aggregation

## CI/CD & Deployment

**Hosting:**
- GitHub Pages (static site from repository root)
- Repository: `HSB9912/maple-daily-tracker`
- Deployment: automatic on push to `main` branch

**CI Pipeline:**
- None - no GitHub Actions, no pre-commit hooks, no linting

## Environment Configuration

**Required env vars:**
- None - all configuration is hardcoded in `index.html`

**Hardcoded configuration:**
- `GIST_API`: `https://api.github.com/gists` (line 292)
- `_tk`: GitHub PAT as charCode array (line 293)
- `GIST_FILE`: `maple-tracker-data.json` (line 297)
- `FIXED_GIST_ID`: `ed1c7a037614886df9cf744decb9079a` (line 298)
- `DEFAULT_RESOURCES`: `['매소', '조각', '경험치', '기운', '상주정', '우유', '이슬', '코젬', '달라심볼']` (line 304)

## Webhooks & Callbacks

**Incoming:**
- None

**Outgoing:**
- None

## Rate Limits & Constraints

**GitHub Gist API:**
- Authenticated requests: 5,000 per hour per token
- Gist size limit: 10MB per file (well within limits for JSON tracker data)
- Single Gist approach means all users share one API token's rate limit

---

*Integration audit: 2026-03-25*
