# CLAUDE.md — seyongeo.github.io

Repository-wide context for AI assistants working in this repo. Sub-app specific rules live in `<sub-app>/CLAUDE.md` files (e.g. [golf/CLAUDE.md](golf/CLAUDE.md)) and are read automatically when editing files under that folder.

---

## Project Type

A personal **GitHub Pages** site. All files in this repo are static and deployed as-is from the `main` branch to https://seyongeo.github.io — there is no build step and no package manager **inside this repo**.

Each top-level folder is a self-contained sub-app:

- `/` (root `index.html`) — Workspace landing dashboard
- `golf/` — AMP99 Golf Manager (React 18 + Babel standalone). **Requires the Spring Boot BFF in the sibling `myapp/` project.** See [golf/CLAUDE.md](golf/CLAUDE.md)
- `project-mockup/` — Static HTML project-management UI mockups (vanilla JS)

---

## Sibling Backend Project

The golf sub-app depends on a Spring Boot backend that lives **outside this repo**, in a sibling directory:

- **Path:** `/Users/seyongeo/code folder/myapp/`
- **Stack:** Spring Boot 3.5, Java 17, Maven, H2 file database
- **Package:** `com.example.myapp.golf.*` (controller, services, JPA entities, DTOs)
- **Endpoints:** all under `/api/golf/*` (see [golf/CLAUDE.md](golf/CLAUDE.md) for the contract)
- **Secrets:** `application-local.yml` (gitignored) — Anthropic API key, shared password, admin password
- **Local URL:** `http://localhost:8080`
- **Separate git repo** — `myapp/` has its own `.git`. Do not try to track it from this repo. (`myapp/.gitignore` excludes `seyongeo.github.io/` for the same reason — they are siblings, not parent/child.)

The mockups under `project-mockup/` and the root dashboard remain pure static files with no backend dependency.

---

## Local Development Servers

For full local development of the golf app you need TWO processes running simultaneously:

### 1. Static file server (port 8000)

**Always use port 8000 for this repo's static server.** Port 8080 is reserved for the Spring Boot backend (`myapp`).

```bash
cd "/Users/seyongeo/code folder/myapp/seyongeo.github.io"
python3 -m http.server 8000
```

Then visit:
- Root dashboard: http://localhost:8000/
- Golf app: http://localhost:8000/golf/
- Mockups: http://localhost:8000/project-mockup/

### 2. Spring Boot BFF (port 8080) — required for `golf/`

```bash
cd "/Users/seyongeo/code folder/myapp"
./mvnw spring-boot:run
```

**The golf app does not work without this.** The mockups and root dashboard work without it.

Opening HTML files directly with `file://` also works for quick UI checks of the mockups or root dashboard, but **the golf app's `fetch` calls to the BFF will fail under `file://`** due to CORS — use `http://localhost:8000/golf/`.

---

## Deployment

- Push to `main` → GitHub Pages auto-deploys (usually < 1 minute)
- No CI/CD, no preview environments, no staging
- **Phase 1 caveat:** the live deployed `golf/` does NOT work because it depends on a local-only BFF. Phase 2 will deploy the BFF to a public host, at which point the live golf app comes back online.

---

## Git Workflow

- Main branch: `main`
- Feature branches (when used): `claude/<description>` or `feature/<description>`
- The sibling `myapp/` repo has its own commit history; commits in this repo only cover frontend (golf/, project-mockup/, root dashboard).

---

## Out of Scope (repo-wide)

Do NOT introduce in **this** repo without explicit instruction:

- Build tools (webpack, vite, rollup, esbuild)
- Package managers (npm, yarn, pnpm)
- CSS preprocessors (Sass, Less)
- TypeScript
- A backend server or database **inside this repo** — the backend lives in `myapp/`, not here
- Test frameworks
- CI/CD pipelines
- Reintroducing the deleted `golf/golf-scores.md` or any markdown-as-database scheme — golf data lives in the BFF's H2 database now

Sub-app specific out-of-scope rules (e.g. "React is allowed only inside `golf/`", "no `localStorage` caching of golf game data") live in the respective sub-app CLAUDE.md.
