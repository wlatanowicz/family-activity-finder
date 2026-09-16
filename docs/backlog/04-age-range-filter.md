---
status: todo
---

# 04. Age range filter

## Summary

Add an age range to each activity and let parents filter the browse list by
a single age ("suitable for my 4-year-old"). This is the "age-based
recommendations" line item from the discovery summary's MVP scope, in its
simplest form — a manual age input, not yet tied to a saved child profile
(that personalization layer is task 11, built on top of this).

## Why

Age-appropriateness is the single most requested filter for a parenting
product and doesn't need any other feature to be useful on its own.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `age_min_years: int | None` — nullable (no known lower bound).
  - `age_max_years: int | None` — nullable (no known upper bound).
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional query param `age_years: int | None`. When set, filter to
  rows where `(age_min_years IS NULL OR age_min_years <= age_years)` AND
  `(age_max_years IS NULL OR age_max_years >= age_years)` — i.e. an unset
  bound means "no restriction on that side," not "excludes everything."
  Combine with `AND` alongside the task 03 filters.
- Add validation: reject (`422`, standard FastAPI validation) `age_years`
  outside `0–17`.
- Extend `test_routes.py`: activity with no age bounds matches any
  `age_years`; activity with both bounds only matches inside the range;
  activity with only one bound set is correctly open-ended on the other
  side.

## Data changes

- `make make-migrations` — both new columns nullable, no backfill needed.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx` filter bar: add a `NumberInput` "Child's age" (0–17,
  empty = no filter), wired into the same `useSearchParams`-backed filter
  state as task 03 (`?age=4`).
- Show an age range badge on each card/detail page, e.g. "Ages 3–8" or
  "Ages 3+" or "All ages" when both bounds are null.
- i18n: `filters.age.label`, `activities.ageRange.bounded` (`"Ages {{min}}–{{max}}"`),
  `.minOnly` (`"Ages {{min}}+"`), `.maxOnly` (`"Up to age {{max}}"`),
  `.any` (`"All ages"`) — added to all four locale files.

## Out of scope

- Saved child profiles (task 10) and auto-applying a child's age (task 11).
- Multiple age ranges per activity (e.g. different sessions for different
  ages) — one range per activity is enough for MVP.
- Gender-suitability filtering — a separate dimension, covered by task 14.

## Acceptance criteria

- `GET /api/activities?age_years=4` includes an activity with
  `age_min_years=3, age_max_years=8`, excludes one with
  `age_min_years=10`, and includes one with both bounds null.
- The age input combines correctly with the indoor/outdoor/duration filters
  from task 03 (all present in the query string at once narrows correctly).
- `make check` passes; `cd frontend && npm run build` succeeds.
