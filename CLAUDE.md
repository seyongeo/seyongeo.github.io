# CLAUDE.md — seyongeo.github.io

This file provides context for AI assistants (Claude Code and others) working in this repository.

---

## Project Overview

A **pure static HTML application** for Korean corporate project management. It renders a dark-themed workspace UI with multiple views for tracking multi-phase projects—milestones, staffing, and budgets. There is no backend, no build tool, and no package manager. Everything runs in-browser.

**Domain language:** Korean (project data, labels, and UI copy are in Korean).

---

## Repository Structure

```
/
├── index.html                    # Main landing page — "seyongeo Workspace"
│                                 #   Dual-view toggle: Dashboard | Explorer
├── project-detail.html           # Project detail view (primary)
├── project-detail2.html          # Design variant
├── project-detail3.html          # Design variant
├── project-detail-input.html     # Data entry form (saves to localStorage)
├── project-detail-fields.html    # Form field component template
├── project-detail-spec.md        # Full design/spec document (~26KB)
├── project-detail-data.json      # JSON data schema with Korean field names
├── project-management.html       # Dashboard (legacy, simpler styling)
├── project-management-view.html  # Dashboard variant
├── project-management-w2.html    # Dashboard variant
├── files/                        # Archived earlier versions of HTML files
├── files 2/                      # Duplicate archive
├── files.zip                     # Archived zip
├── test_memo.txt                 # Scratch file (Korean "테스트")
└── .claude/settings.local.json   # Claude Code local permissions
```

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Markup | HTML5 (no templates, no preprocessors) |
| Styling | CSS3 with custom properties, embedded `<style>` tags |
| Scripting | Vanilla JavaScript, embedded `<script>` tags |
| Fonts | Google Fonts CDN — Instrument Sans + JetBrains Mono |
| Data persistence | Browser `localStorage` (key: `pgDetailData`) |
| Deployment | GitHub Pages (static files, no build step) |
| Build tools | **None** |
| Package manager | **None** |

---

## Development Workflow

### Running Locally

Open any HTML file directly in a browser — no server required.

```bash
open index.html
# or
open project-detail.html
```

For live-reload during editing, a simple static server works:

```bash
python3 -m http.server 8080
# Then visit http://localhost:8080
```

### Data Flow

1. User fills out `project-detail-input.html`
2. Form saves JSON to `localStorage` under key `pgDetailData`
3. `project-detail.html` reads `pgDetailData` on load and renders the UI dynamically

### No Build Step

Files are deployed as-is. There is no compilation, bundling, transpiling, or minification pipeline.

---

## Design System

All pages share a consistent dark-theme design system defined via CSS custom properties.

### Color Palette

```css
--color-bg:               #0a0e1a   /* darkest background */
--color-surface:          #141925   /* card surfaces */
--color-surface-elevated: #1c2331   /* elevated elements */
--color-border:           #252d3f   /* borders */
--color-text-primary:     #e4e8f0   /* main text */
--color-text-secondary:   #8b95a8   /* secondary text */
--color-text-tertiary:    #5a6375   /* muted text */
--color-accent:           #4f7cff   /* blue accent */
--color-accent-light:     #6b91ff
--color-success:          #22c55e   /* green */
--color-warning:          #f59e0b   /* amber */
--color-danger:           #ef4444   /* red */
--color-purple:           #a855f7
--color-cyan:             #06b6d4
```

### Typography

- **Primary font:** `Instrument Sans` (weights: 400, 500, 600, 700)
- **Monospace font:** `JetBrains Mono` (weights: 400, 500)

### Layout Pattern

- Fixed sidebar: 280px wide with `slideInLeft 0.5s ease-out` animation
- Fixed header: `slideInDown 0.5s ease-out`
- Content cards: `fadeInUp 0.5s ease-out` with staggered delays
- CSS Grid for card grids; Flexbox for rows

### Animations

```css
slideInLeft   0.5s ease-out   /* sidebar */
slideInDown   0.5s ease-out   /* header */
fadeInUp      0.5s ease-out   /* cards, staggered with animation-delay */
```

---

## Key Conventions

### HTML

- All CSS is embedded in `<style>` tags within each HTML file — no external stylesheets.
- All JavaScript is embedded in `<script>` tags — no external `.js` files.
- Each HTML file is self-contained and independently deployable.
- Prefer duplicating styles/scripts across files over creating shared assets (consistent with existing pattern).

### CSS

- Use CSS custom properties (variables) for all colors and consistent values.
- Do not add utility-class frameworks (Tailwind, Bootstrap, etc.).
- Maintain the dark-theme aesthetic — do not introduce light backgrounds.

### JavaScript

- Vanilla JS only — no frameworks (React, Vue, etc.).
- DOM manipulation via `querySelector`/`querySelectorAll`.
- Data persistence: always use `localStorage` key `pgDetailData`.
- No `fetch`/API calls to external services.

### Korean Language

- All UI labels, data fields, and user-facing text are in Korean.
- JSON data schema uses Korean keys (e.g., `프로젝트명`, `담당자`, `마일스톤_진행현황`).
- Status values: `완료` (complete), `진행중` (in progress), `대기` (waiting).
- Staffing levels: `특급` (expert), `고급` (senior), `중급` (mid), `초급` (junior).

### File Naming

- HTML page variants use numeric suffixes: `project-detail.html`, `project-detail2.html`, `project-detail3.html`.
- Use kebab-case for all file names.

---

## Data Schema

The primary data structure stored in `localStorage`:

```json
{
  "프로젝트_진행_상황_요약": {
    "프로젝트명": "",
    "고객사명": "",
    "계약_시작일": "",
    "계약_종료일": "",
    "우선순위": "",
    "전체_진행률": "",
    "경과_일수": "",
    "남은_일수": "",
    "담당자": ""
  },
  "마일스톤_진행현황": [
    {
      "단계_번호": 1,
      "단계명": "",
      "시작일": "",
      "종료일": "",
      "상태": "",
      "작업_내용": [],
      "단계_진행률": ""
    }
  ],
  "월별_인력_투입_계획_및_실적": [],
  "월별_비용_집행_계획_및_실적": []
}
```

---

## Git Workflow

- **Main branch:** `main`
- **Feature branches:** `claude/<description>` pattern (e.g., `claude/add-claude-documentation-B1rUl`)
- No CI/CD pipelines exist; pushes to `main` deploy automatically via GitHub Pages.
- Commit messages are imperative and descriptive (e.g., "Replace dashboard table with card grid layout").

---

## Reference Documents

- **`project-detail-spec.md`** — Comprehensive visual and functional specification for `project-detail.html`. Consult this before making changes to the detail page layout, sections, or data rendering logic.
- **`project-detail-data.json`** — Canonical JSON schema. Use as the reference for all field names and data structure.

---

## Out of Scope

The following do NOT exist in this project and should not be introduced without explicit instruction:

- Build tools (webpack, vite, rollup, esbuild)
- Package managers (npm, yarn, pnpm)
- CSS preprocessors (Sass, Less)
- JavaScript frameworks (React, Vue, Svelte, etc.)
- TypeScript
- Backend services or APIs
- Test frameworks
- CI/CD pipelines
