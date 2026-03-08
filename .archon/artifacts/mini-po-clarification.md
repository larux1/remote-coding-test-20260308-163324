# Mini-PO Clarification — US-0: Setup projet

**Issue**: #3
**Track**: COMPLEXE
**Date**: 2026-03-08

---

## Interpreted Requirements

US-0 asks for **three deliverables** (matching AC0.1–AC0.3):

### 1. CLAUDE.md (AC0.1)
A root-level `CLAUDE.md` file describing:
- Project overview (bookmark manager with tags, single-user, no auth)
- Tech stack (FastAPI backend, React frontend, Docker Compose deployment)
- Repository conventions (monorepo layout, coding standards)
- Common commands (dev, test, lint, build, docker compose)

### 2. Monorepo scaffolding (AC0.2)
Two top-level directories with minimal but runnable boilerplate:

**`backend/`** — FastAPI project:
- Python project with `pyproject.toml` (or `requirements.txt`)
- FastAPI app entry point with a health-check endpoint
- Test setup (pytest) with at least one passing test
- Linter config (ruff or flake8)

**`frontend/`** — React project:
- JavaScript/React app (Vite or CRA scaffolding)
- Basic `package.json` with dev/build/test/lint scripts
- At least one passing test
- Linter config (ESLint)

### 3. Basic CI pipeline (AC0.3)
A GitHub Actions workflow (`.github/workflows/ci.yml`) that runs on push and triggers:
- Backend: lint + test
- Frontend: lint + test

---

## Assumptions Made

| # | Assumption | Rationale |
|---|-----------|-----------|
| A1 | **JavaScript (not TypeScript)** for frontend | The constraint says "Frontend JavaScript/React" explicitly. Will use plain JS. |
| A2 | **Vite** as React build tool | CRA is deprecated. Vite is the standard choice for new React projects. |
| A3 | **SQLite** for local DB during scaffolding | The spec says "volume Docker ou fichier DB" and no specific DB is mandated for MVP. SQLite keeps scaffolding simple. PostgreSQL can be introduced later with SQLAlchemy. However, the `.mcp.json` references PostgreSQL — will use **SQLAlchemy with SQLite for dev** and leave PostgreSQL migration straightforward. |
| A4 | **Ruff** for Python linting | Modern, fast, covers both linting and formatting. Standard for new Python projects. |
| A5 | **ESLint** for JS linting | Standard for React projects. |
| A6 | **pytest** for backend tests | Standard for FastAPI projects. |
| A7 | **Vitest** for frontend tests | Pairs naturally with Vite, faster than Jest for Vite projects. |
| A8 | **GitHub Actions** for CI | Repo is on GitHub, so GitHub Actions is the natural CI choice. |
| A9 | **Docker Compose** is NOT part of US-0 scope | US-0 only mentions scaffolding + CLAUDE.md + CI. Docker Compose is an NFR (NFR2) but will be addressed in later user stories when there is actual application logic to deploy. A `Dockerfile` per service and `docker-compose.yml` skeleton may be included if time permits, but are not required by AC0.1–AC0.3. |
| A10 | **No application logic** in this US | US-0 is purely infrastructure. The health-check endpoint is the only "functional" code, serving as proof the stack works. |
| A11 | **Python 3.11+** | Modern FastAPI standard. Will target 3.12 in CI. |
| A12 | **Node 20 LTS** | Current LTS for frontend tooling. |

---

## Scope Boundaries

### In scope (US-0)
- `CLAUDE.md` at repo root
- `backend/` directory with FastAPI scaffolding (app, health endpoint, pyproject.toml, pytest config, ruff config)
- `frontend/` directory with React scaffolding (Vite + React, package.json with scripts, ESLint config, basic test)
- `.github/workflows/ci.yml` running lint + test for both backend and frontend
- Update `.gitignore` to cover both stacks properly

### Out of scope (deferred to later US)
- Database models, migrations, actual API endpoints (US-1+)
- Docker Compose setup (can be done as part of US-1 or a dedicated infra task)
- Any bookmark CRUD functionality
- Authentication (explicitly out of scope per spec)
- Import/export (v2 per spec)

---

## Questions for the User

### Non-blocking (assumptions made, override if needed)

1. **Frontend language**: The constraint says "JavaScript/React". Should this be **plain JavaScript** or would you prefer **TypeScript**? (Assumed: plain JS per constraint wording.)

2. **Database for scaffolding**: Should the backend scaffolding include a database setup (SQLAlchemy + SQLite), or just a bare FastAPI app with no DB? (Assumed: include SQLAlchemy skeleton with SQLite so later stories can build on it.)

3. **Docker files in US-0**: Should `Dockerfile` + `docker-compose.yml` be included in the scaffolding, or deferred? (Assumed: deferred, since AC0.1–AC0.3 don't mention Docker.)

4. **Package manager for frontend**: `npm`, `yarn`, or `pnpm`? (Assumed: `npm` as the most standard default.)

### No blocking questions identified
All ambiguities have reasonable defaults based on the spec constraints. Implementation can proceed with the assumptions above.

---

## Proposed Deliverable Summary

```
.
├── CLAUDE.md
├── .github/
│   └── workflows/
│       └── ci.yml
├── backend/
│   ├── pyproject.toml
│   ├── app/
│   │   ├── __init__.py
│   │   └── main.py          # FastAPI app + /health
│   └── tests/
│       ├── __init__.py
│       └── test_health.py
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── eslint.config.js
│   ├── index.html
│   ├── src/
│   │   ├── main.jsx
│   │   └── App.jsx
│   └── tests/
│       └── App.test.jsx
├── .gitignore                # updated
└── README.md                 # keep existing
```
