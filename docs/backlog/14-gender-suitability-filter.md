---
status: todo
---

# 14. Gender-suitability filter

## Summary

Add an optional gender-suitability tag to each activity (e.g. a dance class
marked "girls," a football clinic marked "boys," most activities left
unmarked/"all") and let parents filter the browse list by it. This is the
second half of the "filtrowanie po wieku/płci" (age/gender filtering) idea
from `docs/backlog/IDEAS.md` — the age half already shipped as task 04.

## Why

Task 04 covers age but not gender; `IDEAS.md` calls out both in the same
idea. Some activities in practice are marketed/run as gender-specific
(certain dance, sports, or scouting programs), so surfacing it as a filter
avoids parents finding activities that turn out not to fit their child once
they show up.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `suitable_for_gender: str | None` — `max_length=10`, nullable. Nullable
    means "all genders" (the common case) — do not require every activity
    to set it. Validate against a `Literal`/constant list at the API layer
    (mirroring task 01's `category` approach): `girls`, `boys`, `all`. A
    row can also simply be `None`, treated identically to `all`.
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional query param `gender: Literal["girls", "boys"] | None`. When
  set, return rows where `suitable_for_gender IS NULL OR
  suitable_for_gender IN ('all', <gender>)` — i.e. gender-specific
  activities for the other gender are excluded, but unmarked/"all"
  activities always match. Combine with `AND` alongside every other filter
  (tasks 03–07).
- Extend `test_routes.py`: `gender=girls` includes activities marked
  `girls` and `all` and null, excludes `boys`; no `gender` param returns
  everything unfiltered.

## Data changes

- `make make-migrations` — new nullable column, no backfill needed
  (existing/new rows default to "all" via `None`).
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx` filter bar: add a `SegmentedControl`/`Select` "Suitable
  for" with options "All," "Girls," "Boys" (default "All," which means no
  filter applied — not "show only unmarked activities"), wired into the
  same `useSearchParams`-backed filter state as tasks 03–04 (`?gender=girls`).
- Show a badge on activity cards/detail page only when
  `suitable_for_gender` is set to `girls` or `boys` (omit the badge
  entirely for `all`/null, matching the existing convention from task 09's
  rating badge of not cluttering the common case).
- i18n: `filters.gender.label`, `.all`, `.girls`, `.boys`,
  `activities.genderBadge.girls`, `.boys` — all four locale files.

## Out of scope

- Any inference/auto-tagging of gender suitability from activity name or
  category — it's a plain admin-entered field (task 12 extends the admin
  form to cover it), not a heuristic.
- Non-binary/other categories beyond "girls / boys / all" — kept to the
  three values named in the source idea; revisit if real content needs
  finer granularity.

## Acceptance criteria

- `GET /api/activities?gender=girls` includes activities marked `girls` or
  `all` or with the field unset, and excludes ones marked `boys`.
- The gender filter combines correctly with the age (task 04) and
  indoor/outdoor/duration (task 03) filters in the same query string.
- Cards for activities with `suitable_for_gender` unset/`all` show no
  gender badge.
- `make check` passes; `cd frontend && npm run build` succeeds.

## Follow-up in other tasks

- Task 12 (admin activity management) should add `suitable_for_gender` to
  its create/edit form field set alongside the other tasks 01–07 columns.
