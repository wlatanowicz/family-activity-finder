---
status: todo
---

# 05. City location filter

## Summary

Add a city/region field to activities and let parents filter the browse
list down to a specific city, via a dropdown populated from the activities
actually in the database. This is the first, simplest slice of the
discovery summary's "location-aware search" — a text match on city, no
maps or distance math yet (that's task 06).

## Why

Most parents search "things to do in [my city]," not by geographic radius.
A city filter is useful standalone and is a prerequisite-free step before
the geolocation/distance feature in task 06.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `city: str | None` — `max_length=100`, indexed. Nullable because not
    every activity will have this back-filled immediately after migration.
- `backend/src/apps/activities/routes.py`:
  - Extend `GET /api/activities` with optional query param `city: str | None`
    — exact, case-insensitive match (normalize both sides with
    `func.lower()` or an equivalent) against `Activity.city`. Combine with
    `AND` alongside existing filters.
  - Add `GET /api/activities/cities` — returns
    `{"cities": ["Berlin", "Munich", ...]}`, the distinct, non-null `city`
    values currently in the table, sorted alphabetically. Powers the
    frontend dropdown without hardcoding a city list.
- Extend `test_routes.py`: `city` filter matches case-insensitively;
  `/cities` returns distinct sorted values and omits nulls.

## Data changes

- `make make-migrations` — nullable `city` column, indexed.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx`: fetch `GET /api/activities/cities` on mount, populate a
  `Select` ("All cities" + each returned city), wired into the same
  `useSearchParams` filter state as tasks 03–04 (`?city=Berlin`).
- Show the city on each activity card (alongside the existing address
  line) and on the detail page.
- i18n: `filters.city.label`, `filters.city.all` — added to all four locale
  files.

## Out of scope

- Free-text/fuzzy city search — an exact-match dropdown is enough since the
  list is generated from real data, not user-typed.
- Distance-based "near me" sorting (task 06).

## Acceptance criteria

- `GET /api/activities/cities` reflects only cities that currently have at
  least one activity.
- `GET /api/activities?city=berlin` matches activities with `city="Berlin"`.
- The city filter composes correctly with age/indoor/duration filters from
  prior tasks.
- `make check` passes; `cd frontend && npm run build` succeeds.
