# Investigation — US-0: Setup projet (scaffolding, CLAUDE.md, CI basique)

**Issue**: #3
**Date**: 2026-03-08
**Type**: Feature (greenfield scaffolding)

---

## Impact Analysis

This is a **greenfield scaffolding task**. The repository currently contains only:
- `README.md` (1-line project description)
- `.gitignore` (basic Python/Node ignores)
- `.mcp.json` (MCP server configs — not part of this US)
- `.archon/` (workflow artifacts — not part of this US)

No application code, no CI, no CLAUDE.md exist yet. Everything needs to be created from scratch.

## Affected Files

### Files to CREATE

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Project overview, conventions, commands (AC0.1) |
| `backend/pyproject.toml` | Python project config (dependencies, ruff, pytest) |
| `backend/app/__init__.py` | Package marker |
| `backend/app/main.py` | FastAPI app entry point + `/health` endpoint |
| `backend/tests/__init__.py` | Package marker |
| `backend/tests/test_health.py` | Health endpoint test |
| `frontend/package.json` | Node project config with dev/build/test/lint scripts |
| `frontend/vite.config.js` | Vite configuration |
| `frontend/eslint.config.js` | ESLint flat config |
| `frontend/index.html` | HTML entry point |
| `frontend/src/main.jsx` | React app bootstrap |
| `frontend/src/App.jsx` | Root React component |
| `frontend/tests/App.test.jsx` | Basic component test |
| `.github/workflows/ci.yml` | CI pipeline: lint + test for both stacks |

### Files to MODIFY

| File | Change |
|------|--------|
| `.gitignore` | Expand to cover both stacks more thoroughly (e.g., `.ruff_cache/`, `*.egg-info/`, `.vite/`, `build/`) |

### Files NOT TOUCHED

| File | Reason |
|------|--------|
| `README.md` | Keep as-is per spec |
| `.mcp.json` | Unrelated infrastructure |
| `.archon/*` | Workflow artifacts, not deliverables |

---

## Proposed Approach

### 1. CLAUDE.md (AC0.1)

Create a root `CLAUDE.md` covering:
- **Project description**: Bookmark manager with tags, single-user, no auth
- **Tech stack**: FastAPI (Python 3.12), React/Vite (JS, Node 20), SQLite/SQLAlchemy (dev), PostgreSQL (prod)
- **Repository layout**: Monorepo with `backend/` and `frontend/`
- **Coding conventions**: Ruff for Python, ESLint for JS
- **Common commands**: Listed per sub-project (dev server, test, lint, build)

### 2. Backend scaffolding (AC0.2)

**Stack**: Python 3.12 + FastAPI + SQLAlchemy (skeleton) + pytest + Ruff

- `pyproject.toml` with:
  - `[project]` metadata, dependencies: `fastapi`, `uvicorn[standard]`
  - `[tool.ruff]` config (line-length=88, select rules)
  - `[tool.pytest.ini_options]` config
- `app/main.py`: FastAPI app with a single `GET /health` → `{"status": "ok"}`
- `tests/test_health.py`: Uses `fastapi.testclient.TestClient` to assert health endpoint returns 200 + correct body

**Key decisions**:
- Use `pyproject.toml` (modern standard) rather than `requirements.txt`
- Ruff for both linting and formatting
- No database models yet (out of scope per clarification A10)
- Include `httpx` as test dependency (required by `TestClient` in modern FastAPI)

### 3. Frontend scaffolding (AC0.2)

**Stack**: Node 20 + React 19 + Vite + Vitest + ESLint

- Bootstrap with Vite React template structure (manual, not `create-vite` CLI)
- `package.json` with scripts: `dev`, `build`, `preview`, `test`, `lint`
- `eslint.config.js`: Flat config format (ESLint 9+), with `eslint-plugin-react`
- `vite.config.js`: React plugin, Vitest configuration
- `src/App.jsx`: Minimal component rendering app name
- `tests/App.test.jsx`: Renders App, checks for text presence

**Key decisions**:
- Plain JavaScript, not TypeScript (per constraint A1)
- Vitest over Jest (natural fit with Vite, per A7)
- ESLint flat config (modern standard, ESLint 9+)
- `@testing-library/react` + `jsdom` for component tests

### 4. CI pipeline (AC0.3)

**`.github/workflows/ci.yml`**:
- Trigger: `push` and `pull_request` on all branches
- Two jobs: `backend` and `frontend` (parallel execution)
- **Backend job**: Python 3.12, install deps via `pip install -e ".[dev]"`, run `ruff check .`, `ruff format --check .`, `pytest`
- **Frontend job**: Node 20, `npm ci`, `npm run lint`, `npm test`

### 5. .gitignore update

Expand to include:
- Python: `.ruff_cache/`, `*.egg-info/`, `.pytest_cache/`, `.venv/`
- Node: `build/`, `.vite/`, `coverage/`
- IDE: `.vscode/`, `.idea/`

---

## Risk Areas & Edge Cases

| Risk | Mitigation |
|------|------------|
| **Python dependency versions** | Pin minimum versions in `pyproject.toml` to avoid breakage (e.g., `fastapi>=0.100`) |
| **ESLint config format** | Use flat config (`eslint.config.js`) since ESLint 9+ deprecates `.eslintrc`. Verify plugin compatibility. |
| **Vitest + React Testing Library setup** | Requires `@testing-library/react`, `@testing-library/jest-dom`, `jsdom` — all must be in devDependencies with correct `vite.config.js` test environment setting |
| **CI pip caching** | Use `actions/setup-python` with cache to speed up CI. Similar for Node with `actions/setup-node` + cache. |
| **pyproject.toml dev dependencies** | Use optional dependency group `[project.optional-dependencies] dev = [...]` for test/lint deps |
| **Frontend test runner** | Vitest needs `run` mode in CI (not watch mode). Ensure `npm test` maps to `vitest run`. |

---

## Verification Checklist

After implementation, verify:
1. `cd backend && pip install -e ".[dev]" && ruff check . && ruff format --check . && pytest` — all pass
2. `cd frontend && npm ci && npm run lint && npm test` — all pass
3. `CLAUDE.md` exists at root with project info, conventions, and commands
4. `.github/workflows/ci.yml` exists and is valid YAML
5. Git push triggers CI on GitHub (manual verification post-merge)
