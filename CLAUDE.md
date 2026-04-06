# CLAUDE.md — seyongeo.github.io

This file provides context for AI assistants (Claude Code and others) working in this repository.

---

## Project Overview

A personal **GitHub Pages site** that hosts a landing workspace and multiple self-contained sub-apps. Each sub-app lives in its own folder and is independently deployable. There is no backend, no build tool, and no package manager — everything is static and runs in-browser.

The repo currently contains two active sub-apps:

1. **Workspace dashboard** (`/index.html`) — landing page that links to sub-apps and mockups
2. **AMP99 Golf Manager** (`/golf/`) — a React-based golf score tracker for 서울대 AMP99 월례회

It also keeps a mockup archive at `/project-mockup/` with several design/layout variants of a Korean corporate project management UI.

**Domain language:** Korean (labels, data fields, commit-adjacent documentation). Code identifiers remain in English.

---

## Repository Structure

```
/
├── index.html                    # "seyongeo Workspace" landing dashboard
├── test_memo.txt                 # Scratch file
├── files.zip                     # Legacy archive (kept for now)
├── golf/                         # ── Sub-app: AMP99 Golf Manager ──
│   ├── index.html                #   React 18 + Babel standalone SPA
│   └── golf-scores.md            #   Single source of truth for members & rounds
├── project-mockup/               # ── Static HTML design variants ──
│   ├── index.html
│   ├── project-detail.html            # Primary detail page
│   ├── project-detail2.html           # Variant
│   ├── project-detail3.html           # Variant
│   ├── project-detail-input.html      # localStorage data-entry form
│   ├── project-detail-fields.html     # Form field component template
│   ├── project-detail-spec.md         # Design/spec (~26KB)
│   ├── project-detail-data.json       # JSON schema (Korean keys)
│   ├── project-management.html        # Dashboard (legacy)
│   ├── project-management-view.html   # Dashboard variant
│   └── project-management-w2.html     # Dashboard variant
└── .claude/                      # Claude Code local config
```

> **Removed (do not recreate):** `files/`, `files 2/` — earlier HTML archive folders that were deleted. The contents were duplicates of files now living under `project-mockup/`.

---

## Technology Stack

| Area | Tech |
|------|------|
| Hosting | GitHub Pages (static files, no build step) |
| Markup / styling | HTML5 + CSS3 with custom properties, embedded `<style>` |
| Root dashboard + mockups | Vanilla JavaScript, embedded `<script>` |
| Golf sub-app | **React 18 + Babel standalone** (CDN), JSX transformed in-browser via `<script type="text/babel">` |
| Fonts | Google Fonts CDN — Instrument Sans + JetBrains Mono (root + mockups); Apple SD Gothic Neo / Noto Sans KR (golf) |
| Persistence | Mixed — see "Data Flow" section below |
| Build tools | **None** |
| Package manager | **None** |
| Tests / CI | **None** |

> **React is allowed only inside `golf/`.** The root dashboard and everything under `project-mockup/` must remain framework-free. When adding a new sub-app, pick one style and stay consistent inside that sub-app's folder.

---

## Development Workflow

### Running Locally

Open any HTML file directly in a browser — no server required:

```bash
open index.html              # root dashboard
open golf/index.html         # golf app
open project-mockup/project-detail.html
```

For consistent relative-path behavior (e.g. `golf/index.html` fetching `./golf-scores.md`), use a simple static server:

```bash
python3 -m http.server 8080
# visit http://localhost:8080
```

### Deployment

- Pushing to `main` deploys automatically via GitHub Pages.
- No CI/CD, no preview environments.

---

## Data Flow

This is the single most important thing to get right — **each sub-app has its own persistence model.** Do NOT mix them.

### 1. Golf app (`golf/`)

- **Source of truth:** `golf/golf-scores.md` — a markdown file with two sections (`## 회원 목록` and `## 라운드 기록`) containing pipe-delimited tables. Both members and rounds live here.
- **Load:** on mount, the app `fetch`es `./golf-scores.md?<timestamp>` (cache-busted) and parses it via `parseMD` in [golf/index.html:38](golf/index.html#L38).
- **Save:** `saveMDToGitHub` in [golf/index.html:103](golf/index.html#L103) PUTs the regenerated markdown back to `golf-scores.md` via the **GitHub Contents API** (`https://api.github.com/repos/seyongeo/seyongeo.github.io/contents/golf/golf-scores.md`) using a user-supplied Personal Access Token. The commit message is always `Update golf-scores.md via AMP99 Golf Manager`.
- **Fallback:** if no PAT is set, the app downloads the markdown as a file and prompts the user to commit manually.
- **`localStorage` — secrets only:**
  - `amp99_github_pat` — GitHub PAT for the save path above
  - `amp99_apikey` — Anthropic API key for the photo AI extraction feature
  - **No game data is cached in `localStorage`.** The older `amp99m` / `amp99r` cache keys were removed in the "single source of truth" refactor — do not re-introduce them.
- **Photo score entry:** [golf/index.html:141](golf/index.html#L141) `PhotoModal` calls Anthropic's `/v1/messages` endpoint directly from the browser (`anthropic-dangerous-direct-browser-access: true`) with `model: 'claude-opus-4-5'` to extract scores from a photo of a scoreboard. If unknown member names are detected, they are auto-added to the member list.

### 2. Mockup pages (`project-mockup/`)

- **Persistence:** `localStorage` key **`pgDetailData`**.
- **Flow:**
  1. User fills out `project-detail-input.html`
  2. Form saves JSON to `localStorage.pgDetailData`
  3. `project-detail.html` (and variants) reads `pgDetailData` on load and renders the UI
- No file I/O, no GitHub API, no network calls.

### 3. Root dashboard (`index.html`)

- Pure static navigation. No data persistence of its own.

---

## Design System

The repo has **two distinct visual themes** — don't mix them.

### Theme A: Navy / purple (root dashboard + `project-mockup/`)

CSS custom properties defined on `:root` in [index.html](index.html):

```css
--bg:               #0a0e1a   /* darkest background */
--surface:          #141925
--surface-elevated: #1c2331
--surface-hover:    #1f2840
--border:           #252d3f
--text-primary:     #e4e8f0
--text-secondary:   #8b95a8
--text-tertiary:    #5a6375
--accent:           #4f7cff   /* primary blue */
--accent-light:     #6b91ff
--success:          #22c55e
--warning:          #f59e0b
--danger:           #ef4444
--purple:           #a855f7
--cyan:             #06b6d4
--orange:           #f97316
--pink:             #ec4899
--teal:             #14b8a6
```

- **Fonts:** `Instrument Sans` (400/500/600/700) for UI, `JetBrains Mono` (400/500) for code/metadata
- **Ambient gradients:** radial blobs via `body::before` / `body::after` (navy + purple)
- **Layout idioms:** centered column (`max-width: 1060px`), card grids with `fadeInUp 0.5s ease-out`, `slideInLeft` sidebars, `slideInDown` headers

### Theme B: Dark green (`golf/index.html`)

Defined as a JavaScript object `C` at [golf/index.html:26](golf/index.html#L26) — **not** CSS variables:

```js
const C = {
  bg:"#080f08", surface:"#101a10", card:"#162016", border:"#243424",
  accent:"#4ade80", accentLow:"rgba(74,222,128,0.12)", text:"#dff0df",
  dim:"#4d754d", gold:"#fbbf24", silver:"#94a3b8", bronze:"#b87333",
  pink:"#f9a8d4", blue:"#93c5fd", red:"#f87171",
};
```

- **Fonts:** `Apple SD Gothic Neo`, `Noto Sans KR` (Korean-first, mobile optimized)
- **Inline style objects:** reusable constants near the top of the file (`iSt`, `thSt`, `tdSt`, `cardSt`, `btn(variant)`, `gTag(gender)`) — reuse these instead of writing ad-hoc styles
- **Layout:** mobile-sized single column with a sticky bottom tab bar (5 tabs: 성적조회 / 성적입력 / 회원관리 / 통계 / 설정)

---

## Key Conventions

### HTML

- All CSS is embedded in `<style>` tags within each HTML file — no external stylesheets.
- All JavaScript is embedded in `<script>` tags — no external `.js` files.
- Each HTML file is self-contained and independently deployable.
- Prefer duplicating styles/scripts across files over creating shared assets (consistent with existing pattern).

### CSS (root dashboard + mockups)

- Use CSS custom properties (variables) for all colors and consistent values.
- Do not add utility-class frameworks (Tailwind, Bootstrap, etc.).
- Maintain the dark-theme aesthetic — do not introduce light backgrounds.

### JavaScript

- **Root dashboard + `project-mockup/`:** vanilla JS only, no frameworks. DOM via `querySelector` / `querySelectorAll`.
- **`golf/`:** React 18 functional components with hooks. Render root: `ReactDOM.createRoot(document.getElementById("root")).render(<App/>)`. Small helpers at module top (`hdScore`, `uid`, `avg`, `rankColor`, `parseMD`, `generateMD`).
- Short, single-letter variable names (`C`, `sc`, `nr`, `h`, `m`, `r`) are idiomatic inside `golf/index.html` — keep them if editing that file, don't rename for clarity.
- No bundler / module system. No `import`/`export`. Everything is global within the `<script>` block.

### Korean Language

- All UI labels, data fields, and user-facing text are in Korean.
- Mockup JSON data uses Korean keys (e.g. `프로젝트명`, `담당자`, `마일스톤_진행현황`).
- Status values in mockups: `완료` / `진행중` / `대기`. Staffing levels: `특급` / `고급` / `중급` / `초급`.
- Golf data uses Korean column headers: `스코어`, `신페리오점수`, `버디`, `파`, `보기`, `장타`, `니어`.
- Golf handicap rule: `hdScore = 스코어 - 72` ([golf/index.html:32](golf/index.html#L32)). `신페리오점수` is a separate user-entered value (not derived from `스코어`).

### File Naming

- kebab-case for file names.
- Mockup variants use numeric suffixes: `project-detail.html`, `project-detail2.html`, `project-detail3.html`.

---

## Data Schemas

### Golf — `golf/golf-scores.md` format

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

- `장타` / `니어` are booleans rendered as `O` / `-`
- Member and round IDs are generated client-side via `uid()` — 7-char random base-36 strings
- `parseMD` / `generateMD` are the only functions that should read/write this format — reuse them, don't hand-roll new parsers

### Mockups — `localStorage.pgDetailData`

```json
{
  "프로젝트_진행_상황_요약": {
    "프로젝트명": "", "고객사명": "",
    "계약_시작일": "", "계약_종료일": "",
    "우선순위": "", "전체_진행률": "",
    "경과_일수": "", "남은_일수": "",
    "담당자": ""
  },
  "마일스톤_진행현황": [
    {
      "단계_번호": 1, "단계명": "",
      "시작일": "", "종료일": "",
      "상태": "", "작업_내용": [],
      "단계_진행률": ""
    }
  ],
  "월별_인력_투입_계획_및_실적": [],
  "월별_비용_집행_계획_및_실적": []
}
```

See `project-mockup/project-detail-data.json` for the canonical reference.

---

## Git Workflow

- **Main branch:** `main` (auto-deploys via GitHub Pages)
- **Feature branches:** `claude/<description>` pattern for AI-assisted work (e.g. `claude/add-claude-documentation-B1rUl`)
- Commit messages: imperative, descriptive (e.g. "Replace dashboard table with card grid layout", "Fix 신페리오 점수/핸디 semantics")
- Automated commits from the golf app use a fixed message: `Update golf-scores.md via AMP99 Golf Manager` — these are expected and should not be squashed or rewritten.

---

## Reference Documents

- **`project-mockup/project-detail-spec.md`** — full visual and functional spec for `project-detail.html`. Consult before restructuring the detail page layout or data rendering.
- **`project-mockup/project-detail-data.json`** — canonical schema for `pgDetailData`. Use for all field names in the mockups.
- **`golf/golf-scores.md`** — live data for the golf app; also serves as the schema reference (the format is documented inline at the top of the file).

---

## Out of Scope

The following do NOT exist in this project and should not be introduced without explicit instruction:

- Build tools (webpack, vite, rollup, esbuild)
- Package managers (npm, yarn, pnpm)
- CSS preprocessors (Sass, Less)
- TypeScript
- A backend server or database
- Test frameworks
- CI/CD pipelines
- External JS frameworks in the root dashboard or `project-mockup/` (React is confined to `golf/`)
- Reintroducing `localStorage` caching of golf game data (members / rounds) — `golf-scores.md` is the only source of truth
