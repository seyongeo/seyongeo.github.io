# CLAUDE.md — AMP99 Golf Manager (`golf/`)

This file provides context for AI assistants (Claude Code and others) when working **inside the `golf/` sub-app**. Applies automatically whenever files under `golf/` are being edited.

---

## What This App Is

**AMP99 Golf Manager** — a React-based golf score tracker for 서울대 AMP99 월례회 (monthly meet-up). It lives at [golf/index.html](index.html) and **requires a running Spring Boot BFF backend** to function. There is no longer any client-side or markdown-file persistence.

- Single-page mobile app (sticky bottom tab bar, 5 tabs: 성적조회 / 성적입력 / 회원관리 / 통계 / 설정)
- Tracks members (회원) and rounds (라운드) with per-player scores including 신페리오 handicap, buddies, pars, bogeys, 장타 (longest drive), 니어핀 (nearest to pin)
- Photo-based score entry: uploads a scoreboard image to the BFF, which proxies it to Anthropic's Claude API and returns extracted scores
- All data is stored server-side in an H2 database managed by the BFF

---

## Backend Dependency

This app does NOT work standalone. It requires a Spring Boot BFF running and reachable from the browser.

- **BFF source:** `/Users/seyongeo/code folder/myapp/` (Spring Boot 3.5, Java 17, Maven)
- **Local dev URL:** `http://localhost:8080`
- **BFF package:** `com.example.myapp.golf.*`
- **BFF must be running** before opening the golf app, otherwise data load fails with "BFF URL이 설정되지 않았습니다" or a network error.

To start the full local stack:
1. Terminal 1 — BFF: `cd "/Users/seyongeo/code folder/myapp" && ./mvnw spring-boot:run`
2. Terminal 2 — static server: `cd "/Users/seyongeo/code folder/myapp/seyongeo.github.io" && python3 -m http.server 8000`
3. Browser: `http://localhost:8000/golf/`
4. First-time only: 설정 탭 → BFF URL `http://localhost:8080` + 공유 비밀번호 입력 → 저장

---

## Tech Stack

| Layer | Tech |
|---|---|
| UI | **React 18** via CDN (`react.production.min.js` + `react-dom.production.min.js`) |
| JSX | **Babel standalone** (`babel.min.js`) — JSX is transformed in-browser via `<script type="text/babel">` |
| Styling | **Inline style objects** (no CSS variables, no stylesheet) |
| Fonts | Apple SD Gothic Neo, Noto Sans KR (Korean-first, mobile optimized) |
| Persistence | **Server-side only** — Spring Boot BFF with H2 file database |
| Photo AI | Spring Boot BFF proxies to Anthropic `/v1/messages` (server holds the API key) |
| Build tools | **None** — no bundler, no package manager, no transpiler step |

> **Do not introduce** a build step, npm packages, a CSS file, TypeScript, or a module system. This app must remain a single self-contained HTML file deployable as-is to GitHub Pages.

---

## File Layout (inside `golf/`)

```
golf/
├── index.html        # The entire app — HTML shell + React code in one file
└── CLAUDE.md         # This file
```

> **Removed (do not recreate):** `golf-scores.md`. The single source of truth used to be a markdown file in the repo, written via the GitHub Contents API. That model was replaced by the Spring Boot BFF + H2 database. Do NOT reintroduce `golf-scores.md`, `parseMD`, `generateMD`, `saveMDToGitHub`, or any GitHub PAT logic.

---

## BFF API Contract

The frontend talks to the BFF via these endpoints. All require `X-Shared-Password` header except where noted.

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/api/golf/data` | shared | Load all members + rounds (server returns `{members, rounds}`) |
| `POST` | `/api/golf/members` | shared | Create one member `{id, name, gender}` |
| `PUT` | `/api/golf/members/{id}` | shared | Update name/gender of a member |
| `DELETE` | `/api/golf/members/{id}` | shared | Delete a member |
| `POST` | `/api/golf/rounds` | shared | Create a round + its scores in one transaction |
| `PUT` | `/api/golf/rounds/{id}` | shared | Replace a round (date, course, scores) |
| `DELETE` | `/api/golf/rounds/{id}` | shared | Delete a round |
| `POST` | `/api/golf/analyze` | shared | Proxy a Claude Messages API call. Increments monthly usage. Returns Anthropic's raw response. Returns 429 with `{used, cap}` when monthly cap is hit. |
| `GET` | `/api/golf/usage` | shared | Returns `{yearMonth, used, cap}` for the current month |
| `POST` | `/api/golf/cap` | **admin** | Update monthly cap. Body: `{cap: int}` |
| `DELETE` | `/api/golf/data` | **admin** | Wipe all members and rounds (operational tables stay). Used by 설정 탭 → 데이터 초기화 |

### Field naming gotcha

The BFF uses camelCase (`periScore`, `yearMonth`) but the frontend's render and sort code historically uses `s.hd` everywhere for the 신페리오 score. The App's `normalizeRounds` helper converts between the two on load. **When adding new score fields, follow the same convention**: BFF DTO uses camelCase, frontend keeps the existing short names.

---

## Frontend State Model

`App` keeps these key pieces of state:

- `members` — persisted members from BFF
- `rounds` — persisted rounds from BFF (already `normalizeRounds`-mapped, so each score uses `s.hd` not `s.periScore`)
- `pendingNewMembers` — **photo-detected new members that have NOT been persisted to BFF yet.** They live here only until the user clicks 저장 in EntryTab. 취소/초기화 discards them. This is the safety mechanism that prevents the photo modal from silently mutating the member master.
- `sc` — current score-entry overlay (`{memberId → {score, hd, buddies, pars, bogeys, longest, nearest, on}}`)
- `tab` — active tab (view/entry/members/stats/settings)

### Critical invariant: photo doesn't mutate master directly

When the photo modal detects a name not in `members`, it does NOT POST to BFF. It pushes the candidate into `pendingNewMembers` and the entry tab shows it as a temporary row. Only the round-save flow (`saveRound`) actually POSTs new members to BFF, and only those whose row survived in `mergedSc`. Cancelling the entry tab erases all pending members without server contact.

If you ever change `applyPhoto`, **do not reintroduce a direct `bffFetch('/api/golf/members', POST)` call**. That would silently create members the user never confirmed.

### Auto-reload on members tab

`App` has a `useEffect([tab])` that calls `reloadData()` whenever the user switches to the `members` tab. This is a safety net so the canonical member list is always fresh when the user opens that tab — even if some other path mutated the master.

---

## EntryTab Layout

The score entry tab has unusual UX because of the photo-detection workflow:

- All rows are **read-only by default** (값 표시되지만 입력 비활성)
- Each row has a **leading checkbox**
- Action buttons:
  - **저장** — persist round + any pending new members + any name edits
  - **수정 / 수정 완료** — toggle inline edit mode for checked rows (이름 + 성별 + 모든 점수 필드)
  - **삭제** — remove checked rows from the current entry (does NOT delete the member from BFF)
  - **+ 참가자 추가** — append a blank row in edit mode
  - **초기화** — clear date/course/sc/pendingNewMembers and start fresh
  - **취소** — discard everything, return to view tab

The button label flips between "수정" and "수정 완료" based on whether all checked rows are currently in edit mode. See `toggleEditMode` and `checkedAreEditing` in `EntryTab`.

### `effectiveMembers` inside EntryTab

EntryTab computes `effectiveMembers = [...members, ...pendingNewMembers]` and uses this for display lookups (so photo-detected members appear in score rows even before they're persisted). It does NOT use this for the master-level operations — only display.

---

## Settings Tab (`SettingsTab`)

Three cards:

1. **🔌 BFF 연결** — BFF server URL + shared password (stored in `localStorage` as `amp99_bff_url`, `amp99_bff_pass`). On save, dispatches a `amp99-bff-changed` window event so `App` reloads data with the new connection.
2. **📊 사용량 / 한도** — current month usage display + cap update form (admin password required for the cap change)
3. **⚠️ 데이터 초기화** — destructive wipe of all game data. Two-step confirmation (first click → "삭제 확정"; second click → actual `DELETE /api/golf/data`). Requires admin password.

### `localStorage` keys

Only two are used:

- `amp99_bff_url` — BFF base URL (e.g. `http://localhost:8080`)
- `amp99_bff_pass` — shared password for non-admin endpoints

> **Removed (do not reintroduce):** `amp99_apikey`, `amp99_github_pat`, `amp99m`, `amp99r`. Legacy keys from the BYOK / markdown era. The App's `useEffect` mount silently calls `localStorage.removeItem` on `amp99_apikey` and `amp99_github_pat` to clean them up from existing browsers.

---

## Photo AI Extraction (`PhotoModal`)

Located in [index.html](index.html) — search for `function PhotoModal`.

- Builds a Claude Messages API request body with the image as base64 + a Korean prompt explaining the AMP99 scoreboard structure
- Sends it via `bffFetch('/api/golf/analyze', POST)` — the BFF adds the API key and forwards to Anthropic
- Parses the Anthropic response (`data.content[0].text`) for a JSON object with `scores` and `perio` arrays
- Matches each extracted row to existing members with **greedy claim** (each member id can only be matched once — handles homonyms like the two 김정섭 in AMP99). Unmatched rows become entries in `pendingNewMembers`.
- The greedy claim is in `findFreeMember` inside `analyze()`. **Do not revert** to the old `members.find(...)` first-match logic — it silently drops rows when there are duplicate names.

### Friendly error messages

`bffFetch` translates BFF responses:
- `401` → "BFF 비밀번호가 잘못되었습니다"
- `429` → "이번 달 한도 초과 (used/cap)"

---

## Design System (dark green theme)

All colors and font-styling are defined as a **JavaScript object `C`** near the top of [index.html](index.html), not as CSS variables:

```js
const C = {
  bg:"#080f08", surface:"#101a10", card:"#162016", border:"#243424",
  accent:"#4ade80", accentLow:"rgba(74,222,128,0.12)", text:"#dff0df",
  dim:"#4d754d", gold:"#fbbf24", silver:"#94a3b8", bronze:"#b87333",
  pink:"#f9a8d4", blue:"#93c5fd", red:"#f87171",
};
```

### Reusable style constants (also near the top)

Reuse these instead of writing ad-hoc inline styles:

- `iSt` — input style
- `thSt` — table header cell style
- `tdSt` — table body cell style
- `cardSt` — card container style
- `btn(variant)` — button factory. Variants: `"p"` (primary/green), `"r"` (red/danger), `"g"` (green-tinted ghost), default (neutral)
- `gTag(gender)` — small gender badge, handles 남/여

### Layout idioms

- Mobile-sized single column (width constrained by viewport, max 540px)
- Sticky bottom tab bar with 5 buttons, active tab in `C.accent` with a top border
- Ranked lists use `rankColor(r)` — gold/silver/bronze for top 3, neutral for the rest

---

## Code Conventions (inside `index.html`)

### Terse naming is idiomatic — keep it
Short single-letter and 2–3 letter names are deliberate for this file:
- `C` — color palette
- `sc` — score state object
- `nr` — new round
- `h` — hole or row
- `m` — member
- `r` — round
- `iSt`, `thSt`, `tdSt`, `cardSt` — style objects

**Do not** rename these to "clearer" long forms when editing unrelated code. Only expand names when introducing genuinely new concepts.

### React patterns
- Functional components with hooks (`useState`, `useEffect`, `useRef`)
- Render root: `ReactDOM.createRoot(document.getElementById("root")).render(<App/>)`
- No `import`/`export`; everything is global inside the single `<script type="text/babel">` block
- No `useCallback`/`useMemo` unless there is a measurable reason — small components, premature optimization is discouraged here

### Small helpers at module top
- `hdScore(s)` → `s - 72` — handicap score calculation (**do not confuse with 신페리오점수**, which is user-entered)
- `uid()` → 7-char random base-36 string for member/round IDs
- `avg(array)` → average
- `rankColor(rank)` → color for top-3 rank badges
- `bffBase()`, `bffPass()`, `bffFetch(path, opts)` — the only sanctioned helpers for talking to the BFF. **Do not hand-roll alternative `fetch` calls** with the BFF URL.

---

## Korean Language / Domain Rules

All UI text, labels, and data fields are in Korean. Column headers in the data and UI:

| Column | Meaning |
|---|---|
| 스코어 | Gross score (total strokes) |
| 신페리오점수 | New Peoria adjusted score — **user-entered**, not derived |
| 버디 | Number of birdies |
| 파 | Number of pars |
| 보기 | Number of bogeys |
| 장타 | Longest drive (boolean: `O` / `-`) |
| 니어 | Nearest to pin (boolean: `O` / `-`) |

### Handicap semantics — this has been miswired before
- `hdScore = 스코어 - 72` — a simple derived value
- `신페리오점수` — a **separate number** the user enters manually per round. It is NOT computed from 스코어 and should never be overwritten by a derived value
- Commits `547e6d4` ("Fix 신페리오 점수/핸디 semantics") and `ea02651` ("Fix 핸디 = 스코어 - 72") fixed this in earlier history. Re-read them before touching score calculations.

### Other Korean values
- Gender: `남` / `여` (drives tag colors via `gTag`)
- Tab labels: `성적조회` / `성적입력` / `회원관리` / `통계` / `설정`

---

## Git Workflow

- Main branch: `main` (auto-deploys via GitHub Pages to `https://seyongeo.github.io/golf/`)
- **Phase 1 caveat:** the live deployed site is currently broken because it requires the BFF, which is local-only. Phase 2 will deploy the BFF to a public host. Until then, only `http://localhost:8000/golf/` (with the BFF running on 8080) actually works.
- Frontend commits live in this repo (`seyongeo.github.io`)
- Backend commits live in the sibling `myapp/` repo (separate `git init`)

---

## Out of Scope

Do NOT introduce any of the following without explicit instruction:

- Bundlers, transpilers, or a build step
- npm / yarn / pnpm packages
- TypeScript
- External CSS files or CSS preprocessors
- JS frameworks other than React 18 (no Vue, Svelte, Next.js, etc.)
- A separate frontend backend, database, or auth provider — the BFF is the only backend
- Test frameworks
- **Reintroducing `golf-scores.md`, `parseMD`, `generateMD`, `saveMDToGitHub`, or any GitHub Contents API call**
- **Reintroducing `localStorage` caching of game data** (members, rounds). Only `amp99_bff_url` and `amp99_bff_pass` are allowed in localStorage.
- **Reintroducing direct `api.anthropic.com` calls from the browser.** All Anthropic traffic must go through `/api/golf/analyze`.
- Reintroducing the GitHub PAT card or `amp99_github_pat` localStorage key
- Calling `bffFetch('/api/golf/members', POST)` from `applyPhoto` — that's the bug the `pendingNewMembers` design exists to prevent
- Renaming the `bffFetch` helper or the BFF endpoint paths (the BFF contract is shared with `myapp/`)
