# Investigation — US-1: Créer un bookmark avec titre, URL et tags

**Issue**: #4
**Date**: 2026-03-08
**Type**: Feature (new CRUD endpoint + database schema + tag association)
**Branch**: `issue-4` (based on `main`, 1 commit)

---

## Current State

### Branch `issue-4` — Starting Point

The `issue-4` branch is based on `main` (initial commit) and contains only:
- `.gitignore`
- `README.md`

The scaffolding from `issue-3` (FastAPI backend, React frontend, CI) has **not been merged** into `main` or `issue-4`. This means this feature branch must either:
1. **Merge/rebase onto `issue-3`** to inherit the scaffolding, OR
2. **Build everything from scratch**, including the scaffolding

**Recommendation**: Merge `issue-3` into `issue-4` first, then build the bookmark feature on top.

### Branch `issue-3` — Available Scaffolding

The `issue-3` branch contains the full project scaffolding:

| Component | Status | Key Files |
|-----------|--------|-----------|
| FastAPI backend | Done | `backend/app/main.py` (8 lines, `/health` endpoint only) |
| Backend config | Done | `backend/pyproject.toml` (fastapi>=0.100, uvicorn, pytest, httpx, ruff) |
| Backend tests | Done | `backend/tests/test_health.py` (2 tests) |
| React frontend | Done | `frontend/src/App.jsx`, `frontend/src/main.jsx` |
| Frontend config | Done | `frontend/package.json`, `frontend/vite.config.js`, `frontend/eslint.config.js` |
| Frontend tests | Done | `frontend/tests/App.test.jsx` (1 test) |
| CI pipeline | Done | `.github/workflows/ci.yml` (backend + frontend jobs) |
| Documentation | Done | `CLAUDE.md` (tech stack, conventions, commands) |

### What Does NOT Exist Yet (anywhere)

- **No database layer**: No SQLAlchemy, no models, no DB config, no migrations
- **No Pydantic schemas**: No request/response models
- **No CRUD endpoints**: Only `/health` exists
- **No tag system**: No tag models or association tables
- **No Docker Compose**: Referenced in spec but not implemented

---

## Spec Analysis

**Source**: `.archon/specs/2.json` (on `issue-2` branch)

### Acceptance Criteria for US-1

| AC | Description | Scope |
|----|-------------|-------|
| AC1.1 | Create bookmark with URL, title, and tags → saved with tags | Backend API + Frontend form |
| AC1.2 | Create bookmark without tags → saved without tags | Backend API (tags optional) |
| AC1.3 | Duplicate URL → rejection with error message | Backend validation (unique URL constraint) |
| AC1.4 | Missing URL → rejection with error message | Backend validation (URL required) |

### Functional Requirements

| FR | Description | Implementation |
|----|-------------|----------------|
| FR1 | Create bookmark with URL, title, optional tags | `POST /api/bookmarks` endpoint |
| FR5 | Associate one or more tags on creation | Many-to-many Bookmark↔Tag |
| FR7 | Auto-create tag on first use | Get-or-create logic in bookmark creation |

### Constraints

- C1: Backend Python/FastAPI
- C4: Single-user, no authentication
- C5: REST API with OpenAPI auto-generated docs
- C6: Monorepo structure (`backend/`, `frontend/`)

### Tech Stack (from CLAUDE.md)

- **Backend**: FastAPI, Python 3.12, SQLAlchemy, Ruff
- **Frontend**: React/Vite, JavaScript, Node 20, ESLint
- **Database**: SQLite (dev), PostgreSQL (production)

---

## Architecture Design

### Database Schema

```
bookmarks
├── id          INTEGER PRIMARY KEY AUTOINCREMENT
├── url         TEXT NOT NULL UNIQUE
├── title       TEXT NOT NULL
├── created_at  DATETIME DEFAULT CURRENT_TIMESTAMP
└── updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP

tags
├── id          INTEGER PRIMARY KEY AUTOINCREMENT
└── name        TEXT NOT NULL UNIQUE

bookmark_tags (association table)
├── bookmark_id INTEGER FK → bookmarks.id
└── tag_id      INTEGER FK → tags.id
└── PRIMARY KEY (bookmark_id, tag_id)
```

### API Design

```
POST /api/bookmarks
  Request Body:
    {
      "url": "https://example.com",      # required
      "title": "Example",                 # required
      "tags": ["dev", "tools"]            # optional, defaults to []
    }

  Response 201:
    {
      "id": 1,
      "url": "https://example.com",
      "title": "Example",
      "tags": ["dev", "tools"],
      "created_at": "2026-03-08T16:00:00Z",
      "updated_at": "2026-03-08T16:00:00Z"
    }

  Response 422 (validation error):
    { "detail": [{"loc": ["body", "url"], "msg": "Field required", ...}] }

  Response 409 (duplicate URL):
    { "detail": "A bookmark with this URL already exists" }
```

### Backend File Structure (new/modified)

```
backend/
├── app/
│   ├── __init__.py          # existing
│   ├── main.py              # MODIFY: add router, DB init
│   ├── database.py          # NEW: SQLAlchemy engine, session, Base
│   ├── models.py            # NEW: Bookmark, Tag, bookmark_tags ORM models
│   ├── schemas.py           # NEW: Pydantic request/response schemas
│   └── routers/
│       ├── __init__.py      # NEW
│       └── bookmarks.py     # NEW: POST /api/bookmarks endpoint
├── tests/
│   ├── __init__.py          # existing
│   ├── conftest.py          # NEW: test DB fixture, test client
│   ├── test_health.py       # existing (no changes)
│   └── test_bookmarks.py    # NEW: tests for bookmark creation
└── pyproject.toml           # MODIFY: add sqlalchemy dependency
```

### Dependencies to Add

- `sqlalchemy>=2.0` — ORM and database engine
- `pydantic[email]` — already included via FastAPI, but schemas need defining

No additional dependencies needed for SQLite (included in Python stdlib).

---

## Affected Files

### Files to Create

| File | Purpose | Lines (est.) |
|------|---------|--------------|
| `backend/app/database.py` | SQLAlchemy engine, SessionLocal, Base declarative base | ~25 |
| `backend/app/models.py` | Bookmark, Tag, bookmark_tags ORM models | ~40 |
| `backend/app/schemas.py` | BookmarkCreate, BookmarkResponse, TagResponse Pydantic models | ~30 |
| `backend/app/routers/__init__.py` | Package marker | 0 |
| `backend/app/routers/bookmarks.py` | POST /api/bookmarks endpoint with get-or-create tag logic | ~45 |
| `backend/tests/conftest.py` | Test DB setup, session override, test client fixture | ~30 |
| `backend/tests/test_bookmarks.py` | Tests covering AC1.1–AC1.4 | ~70 |

### Files to Modify

| File | Change | Lines affected |
|------|--------|---------------|
| `backend/app/main.py` | Add DB startup event, include bookmarks router | ~10 new lines |
| `backend/pyproject.toml` | Add `sqlalchemy>=2.0` to dependencies | 1 line |

---

## Implementation Approach

### Step 1: Merge issue-3 scaffolding
Merge `issue-3` branch into `issue-4` to get the full project scaffolding.

### Step 2: Add SQLAlchemy dependency
Add `sqlalchemy>=2.0` to `backend/pyproject.toml`.

### Step 3: Create database layer
- `database.py`: Engine (SQLite for dev), sessionmaker, declarative Base
- Use `sqlite:///./bookmarks.db` as the default database URL

### Step 4: Create ORM models
- `models.py`: Bookmark model, Tag model, bookmark_tags association table
- Unique constraint on `bookmarks.url`
- Unique constraint on `tags.name`

### Step 5: Create Pydantic schemas
- `schemas.py`: BookmarkCreate (input), BookmarkResponse (output), TagResponse
- Tags as `list[str]` in both input and output for simplicity

### Step 6: Create bookmark router
- `routers/bookmarks.py`: `POST /api/bookmarks`
- Get-or-create logic for tags (FR7)
- Handle duplicate URL with 409 Conflict (AC1.3)
- Validation via Pydantic (AC1.4 — URL required)

### Step 7: Wire up in main.py
- Import and include the bookmarks router
- Add DB table creation on startup (using `Base.metadata.create_all`)
- Add dependency injection for DB session

### Step 8: Write tests
- `conftest.py`: In-memory SQLite test database, session override
- `test_bookmarks.py`: One test per AC (AC1.1–AC1.4), plus edge cases

### Step 9: Verify
- Run `ruff check .` and `ruff format --check .`
- Run `pytest` — all tests must pass
- Verify OpenAPI docs are auto-generated at `/docs`

---

## Risk Areas and Edge Cases

| Risk | Severity | Mitigation |
|------|----------|------------|
| `issue-4` not based on `issue-3` | **High** | Merge issue-3 first; if conflicts arise, resolve carefully |
| SQLite concurrent writes | Low | Single-user app, no concurrency issue |
| Tag name normalization | Medium | Lowercase and strip whitespace on tag names to prevent near-duplicates |
| URL validation depth | Medium | Use Pydantic `HttpUrl` or basic string validation; don't over-validate |
| Database file location | Low | Use relative path `./bookmarks.db`, add to `.gitignore` |
| Empty title handling | Low | Title is required per schema; Pydantic enforces non-empty |
| Large tag lists | Low | No limit specified; reasonable for single-user |
| Existing test breakage | Low | Health tests are isolated; new DB setup shouldn't affect them |
| Missing `package-lock.json` | Medium | Issue-3 should have it; verify after merge |

---

## Test Plan

| Test | AC | Expected |
|------|----|----------|
| `test_create_bookmark_with_tags` | AC1.1 | 201, bookmark returned with tags `["dev", "tools"]` |
| `test_create_bookmark_without_tags` | AC1.2 | 201, bookmark returned with empty tags `[]` |
| `test_create_bookmark_duplicate_url` | AC1.3 | 409, error message about duplicate URL |
| `test_create_bookmark_missing_url` | AC1.4 | 422, validation error for missing URL |
| `test_create_bookmark_tags_auto_created` | FR7 | Tags created in DB when used for first time |
| `test_create_bookmark_reuses_existing_tags` | FR7 | Existing tags reused, no duplicates created |
| `test_health_still_works` | — | Existing health tests continue to pass |
