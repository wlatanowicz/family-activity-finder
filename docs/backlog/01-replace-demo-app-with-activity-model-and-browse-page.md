---
status: todo
---

# 01. Replace demo app with Activity model and browse page

## Summary

Remove the template's placeholder `items` demo (backend `demo` app + frontend
items list) and replace it with the real domain model for this product:
`Activity`. Ship the first real screen — a public "Browse Activities" page
that lists every activity in the database.

## Why

Every later task (filters, favorites, ratings, child profiles) needs an
`Activity` table and a list endpoint to build on. The discovery summary's MVP
starts with "help parents discover activities" — this task is the smallest
slice that makes that literally true, even before any filtering exists.

## Backend changes

- Delete `backend/src/apps/demo/` (models, routes, api_errors, `__init__.py`).
- Remove `demo_router` import/`include_router` call in `backend/src/main.py`.
- Delete the demo test in `backend/tests/test_main.py`
  (`test_list_items_without_database`, and the `/api/items` assertions).
- Add `backend/src/apps/activities/` package:
  - `models.py` — `Activity` SQLModel table `activities`:
    - `id: UUID` — primary key, `default_factory=uuid4`
    - `name: str` — `max_length=255`, indexed
    - `description: str` — free text (use `sa_column=Column(Text)`)
    - `address: str` — free-text address/venue line, `max_length=500`
    - `category: str` — `max_length=50`, indexed. Not a DB enum: the
      taxonomy will keep growing and shouldn't require a migration each
      time. Validate against a `Literal`/constant list at the API layer
      instead (suggested starter set: `outdoor_play`, `museum_culture`,
      `sports`, `nature_hiking`, `arts_crafts`, `entertainment`,
      `water_activities`, `other`).
    - `created_at: datetime` — `Column(DateTime(timezone=True))`,
      server-set default (`default_factory=lambda: datetime.now(UTC)`)
  - `api_errors.py` — `ApiErrorCode` `StrEnum` with `activity_not_found`
    (used starting in task 02, define it now for consistency with other
    apps' `api_errors.py` files).
  - `routes.py` — `APIRouter(prefix="/api/activities", tags=["activities"])`:
    - `GET /api/activities` — returns
      `{"activities": [{id, name, description, address, category}, ...]}`.
      When `get_db_session` yields `None`, return
      `{"activities": [], "detail": "database not configured", "detail_code": "database_not_configured"}`
      (same shape the demo app used, so `App.tsx` error handling carries
      over).
  - `__init__.py`
- Register `activities_router` in `backend/src/main.py` in place of
  `demo_router`.

## Data changes

- Generate the Alembic migration inside the running stack:
  `make make-migrations` (autogenerate), then review the generated file —
  it should `drop_table("items")` and `create_table("activities")`.
- Apply with `make migrate`.
- No seed data yet — an empty list is an acceptable (if unexciting) result
  for this task; task 12 gives the team a way to populate real activities.

## Frontend changes

- `frontend/src/App.tsx`:
  - Remove the `ItemsResponse` type, `items` state, and the `/api/items`
    fetch + rendering block.
  - Add an `Activity` type (`{id, name, description, address, category}`)
    and `activities` state, fetched from `GET /api/activities` the same way
    `items` was fetched.
  - Render an "Browse Activities" `Paper` with a `List`/card per activity
    showing name, category `Badge`, address, and a truncated description.
    Empty state: "No activities yet."
- `frontend/src/i18n/locales/en.json`: replace the `items` key with an
  `activities` key (`title`, `empty`), and replace
  `errors.itemsRequestFailed` with `errors.activitiesRequestFailed`. Mirror
  the same key renames in `de.json`, `pl.json`, `uk.json` (translate the
  strings — don't leave English text under those locales).

## Out of scope

- Filtering, sorting, pagination (tasks 03–07).
- Activity detail view and routing (task 02).
- Anything for creating/editing activities (task 12).

## Acceptance criteria

- `GET /api/items` and the `demo` app no longer exist anywhere in the repo.
- `GET /api/activities` returns `{"activities": []}` against a fresh,
  migrated database.
- Creating a row directly in the `activities` table makes it appear in the
  browse page on next load.
- `make check` passes (tests, lint, lambda requirements export).
