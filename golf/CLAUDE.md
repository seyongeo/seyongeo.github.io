# CLAUDE.md — AMP99 Golf Manager (`golf/`)

This file provides context for AI assistants (Claude Code and others) when working **inside the `golf/` sub-app**. Applies automatically whenever files under `golf/` are being edited.

---

## What This App Is

**AMP99 Golf Manager** — a React-based golf score tracker for 서울대 AMP99 월례회 (monthly meet-up). It lives at [golf/index.html](index.html) and persists data to [golf/golf-scores.md](golf-scores.md).

- Single-page mobile app (sticky bottom tab bar, 5 tabs: 성적조회 / 성적입력 / 회원관리 / 통계 / 설정)
- Tracks members (회원) and rounds (라운드) with per-player scores including 신페리오 handicap, buddies, pars, bogeys, 장타 (longest drive), 니어핀 (nearest to pin)
- Supports photo-based score entry via Anthropic API (sends a scoreboard image to Claude and extracts structured data)

---

## Tech Stack

| Layer | Tech |
|---|---|
| UI | **React 18** via CDN (`react.production.min.js` + `react-dom.production.min.js`) |
| JSX | **Babel standalone** (`babel.min.js`) — JSX is transformed in-browser via `<script type="text/babel">` |
| Styling | **Inline style objects** (no CSS variables, no stylesheet) |
| Fonts | Apple SD Gothic Neo, Noto Sans KR (Korean-first, mobile optimized) |
| Data format | Markdown tables (`golf-scores.md`) |
| Persistence | GitHub Contents API (via user-supplied PAT) + client-side `fetch` |
| Photo AI | Anthropic `/v1/messages` API, direct browser call |
| Build tools | **None** — no bundler, no package manager, no transpiler step |

> **Do not introduce** a build step, npm packages, a CSS file, TypeScript, or a module system. This app must remain a single self-contained HTML file deployable as-is to GitHub Pages.

---

## File Layout (inside `golf/`)

```
golf/
├── index.html        # The entire app — HTML shell + React code in one file
└── golf-scores.md    # Data file — members and rounds (single source of truth)
```

---

## Running Locally

For the app's relative `fetch('./golf-scores.md')` to work, serve via a static server instead of `file://`:

```bash
python3 -m http.server 8080
# then visit http://localhost:8080/golf/
```

Opening `index.html` directly with `open` also works for UI testing but the data load will fail due to CORS on `file://`.

---

## Data Flow — Read This Before Changing Persistence

**`golf-scores.md` is the single source of truth.** There is no localStorage cache of game data. Do not reintroduce the old `amp99m` / `amp99r` localStorage keys — they were deliberately removed in the "single source of truth" refactor.

### Load (on app mount)
1. `fetch('./golf-scores.md?' + Date.now())` — the timestamp is cache-busting, keep it
2. `parseMD(text)` — located near the top of [index.html](index.html). Reads the `## 회원 목록` and `### round: <id>` sections into `{members, rounds}`
3. State is populated via `setMembers` / `setRounds`. If the fetch or parse fails, show an error — do not silently fall back to sample data

### Save (after any edit)
1. `generateMD(members, rounds)` serializes state back to the markdown format
2. `saveMDToGitHub` PUTs the result via the GitHub Contents API:
   - Endpoint: `https://api.github.com/repos/seyongeo/seyongeo.github.io/contents/golf/golf-scores.md`
   - Requires the current file SHA (fetch via GET first)
   - Commit message is always the literal string `Update golf-scores.md via AMP99 Golf Manager` — **do not change this**; downstream tooling may filter by it
3. If no PAT is set (`localStorage.amp99_github_pat` missing), fall back to downloading the `.md` file locally and alerting the user to commit manually
4. Call `triggerSave()` after every mutation (add/edit/delete member, save round, delete round)
5. Surface the save state in the header: `saving` / `ok` / `error`

### localStorage — secrets only
Only two keys are allowed:
- `amp99_github_pat` — GitHub Personal Access Token (for the save path above)
- `amp99_apikey` — Anthropic API key (for photo AI extraction)

Both are managed in the **설정 (Settings)** tab. Never store any other data in localStorage.

---

## Photo AI Extraction (`PhotoModal`)

Located in [index.html](index.html) (search for `function PhotoModal`).

- Sends the uploaded image (base64) to `https://api.anthropic.com/v1/messages`
- Required header: `anthropic-dangerous-direct-browser-access: true` (this is what permits direct browser → Anthropic API calls; don't remove it)
- Model: `claude-opus-4-5`
- The prompt must explain the complex scoreboard table structure — names can appear in multiple columns, and `hd` (신페리오 핸디) needs to be matched by name across columns, not by row position. This is a recurring source of bugs; previous commits like `23dd594`, `2f693fe`, and `aa21e96` fixed variants of it — consult those commits before rewriting the prompt
- **Auto-add new members:** if the AI detects a name that is not in the current member list, the app adds a new member entry for that person automatically (existing behavior — preserve it)

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

- Mobile-sized single column (width constrained by the viewport)
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
- `parseMD(text)` / `generateMD(members, rounds)` — the only sanctioned reader/writer for `golf-scores.md`. **Do not hand-roll alternative parsers**

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
- Commits `547e6d4` ("Fix 신페리오 점수/핸디 semantics") and `ea02651` ("Fix 핸디 = 스코어 - 72") fixed this. Re-read them before touching score calculations

### Other Korean values
- Gender: `남` / `여` (drives tag colors via `gTag`)
- Tab labels: `성적조회` / `성적입력` / `회원관리` / `통계` / `설정`

---

## Data Schema — `golf-scores.md`

```markdown
# 서울대 AMP99 골프 성적 데이터

> 이 파일은 서울대 AMP99 월례회 성적 데이터입니다.
> golf/index.html 앱이 이 파일을 읽어 데이터를 로드합니다.

---

## 회원 목록

| id | 이름 | 성별 |
|----|------|------|
| <uid> | <name> | 남 or 여 |
...

---

## 라운드 기록

### round: <round_id>
- 날짜: YYYY-MM-DD
- 코스: <course name>

| 회원id | 이름 | 스코어 | 신페리오점수 | 버디 | 파 | 보기 | 장타 | 니어 |
|--------|------|--------|------------|------|----|------|------|------|
| <uid> | <name> | <int> | <number> | <int> | <int> | <int> | O or - | O or - |
...
```

Rules:
- Member and round IDs are 7-char base-36 strings from `uid()` — never hand-write them
- Empty numeric cells in a round should be `0`, not blank
- Boolean cells (`장타`, `니어`) are exactly `O` or `-` (ASCII hyphen, not en-dash)
- Multiple rounds are separated by `### round: ` headers — never combine them into one table

---

## Git Workflow

- Main branch: `main` (auto-deploys via GitHub Pages to `https://seyongeo.github.io/golf/`)
- Automated save commits from the app use the fixed message `Update golf-scores.md via AMP99 Golf Manager` — expected; do not squash or rewrite them
- Manual feature commits should use imperative Korean-or-English descriptions (examples from history: "Fix 신페리오 점수/핸디 semantics", "Add 장타/니어핀 fields to score entry, view, and MD format")

---

## Out of Scope

Do NOT introduce any of the following without explicit instruction:

- Bundlers, transpilers, or a build step
- npm / yarn / pnpm packages
- TypeScript
- External CSS files or CSS preprocessors
- JS frameworks other than React 18 (no Vue, Svelte, Next.js, etc.)
- A backend server or database
- Test frameworks
- **Reintroducing localStorage caching of game data** — the app was deliberately refactored away from this
- Renaming the save commit message string
- Breaking the photo extraction's name-based column matching (revert to row-position matching)
