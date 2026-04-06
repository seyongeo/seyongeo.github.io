# CLAUDE.md — seyongeo.github.io

Repository-wide context for AI assistants working in this repo. Sub-app specific rules live in `<sub-app>/CLAUDE.md` files (e.g. [golf/CLAUDE.md](golf/CLAUDE.md)) and are read automatically when editing files under that folder.

---

## Project Type

A personal **GitHub Pages** site. All files are static and deployed as-is from the `main` branch to https://seyongeo.github.io — there is no build step, no package manager, no backend.

Each top-level folder is a self-contained sub-app:

- `/` (root `index.html`) — Workspace landing dashboard
- `golf/` — AMP99 Golf Manager (React 18 + Babel standalone). See [golf/CLAUDE.md](golf/CLAUDE.md)
- `project-mockup/` — Static HTML project-management UI mockups (vanilla JS)

---

## Local Development Server

**Always use port 8000 for the local static server for this project.** Port 8080 is occupied on this machine by a separate Java/Spring Boot process (`myapp` backend) — do not suggest or use 8080.

```bash
cd "/Users/seyongeo/code folder/myapp/seyongeo.github.io"
python3 -m http.server 8000
```

Then visit:

- Root dashboard: http://localhost:8000/
- Golf app: http://localhost:8000/golf/
- Mockups: http://localhost:8000/project-mockup/

Opening HTML files directly with `file://` also works for quick UI checks, but **`golf/`'s `fetch('./golf-scores.md')` will fail under `file://`** — use the HTTP server whenever live data is needed.

---

## Deployment

- Push to `main` → GitHub Pages auto-deploys (usually < 1 minute)
- No CI/CD, no preview environments, no staging

---

## Git Workflow

- Main branch: `main`
- Feature branches (when used): `claude/<description>` or `feature/<description>`
- Automated commits from the golf app use the fixed message `Update golf-scores.md via AMP99 Golf Manager` — these are expected; do not squash or rewrite them

---

## Out of Scope (repo-wide)

Do NOT introduce without explicit instruction:

- Build tools (webpack, vite, rollup, esbuild)
- Package managers (npm, yarn, pnpm)
- CSS preprocessors (Sass, Less)
- TypeScript
- A backend server or database
- Test frameworks
- CI/CD pipelines

Sub-app specific out-of-scope rules (e.g. "React is allowed only inside `golf/`") live in the respective sub-app CLAUDE.md.
