---
status: todo
---

# 03. Indoor/outdoor and duration filters

## Summary

Add the "indoor/outdoor" and "duration" filters called out in the discovery
summary's MVP scope. Parents can narrow the browse list to activities that
fit whether they're stuck inside or have a specific amount of free time.

## Why

These two are grouped in the discovery summary as a single MVP filter set,
and both are simple scalar columns with no external dependency (unlike
weather or location) — the cheapest filters to ship first.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `is_indoor: bool` — not nullable, default `True` (safe default so
    existing rows don't need backfill logic beyond the default).
  - `duration_minutes: int | None` — approximate typical visit length;
    nullable because not every activity has a fixed duration (e.g. an
    open-ended park).
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional query params:
  - `indoor: bool | None` — when set, filter `is_indoor == indoor`.
  - `max_duration_minutes: int | None` — when set, filter
    `duration_minutes <= max_duration_minutes` (rows with `NULL` duration
    are excluded when this filter is active, since "fits in 30 minutes" is
    unknown for them).
  - Both params combine with `AND` and with each other; no params = no
    filtering (current behavior preserved).
- Update `backend/src/apps/activities/tests/test_routes.py` with cases for
  each param alone and combined.

## Data changes

- `make make-migrations` then review: `add_column("activities", "is_indoor", ...)`
  with `server_default="true"` so the migration backfills existing rows
  without a null gap, and `add_column("activities", "duration_minutes", ...)`
  nullable.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx`: add a filter bar above the list:
  - Indoor/Outdoor/Any — `SegmentedControl` (Mantine), three options.
  - Duration — `Select` with buckets: "Any", "Under 30 min", "30–60 min",
    "1–2 hours", "2+ hours" (map bucket to `max_duration_minutes`: 30, 60,
    120, and "Any"/"2+ hours" sends no param or a high ceiling —
    "2+ hours" should NOT send `max_duration_minutes`, since it means "no
    upper bound", not "under some max"; only the first three buckets map to
    the param).
  - Filter state lives in the URL query string (e.g. `?indoor=true&maxDuration=60`)
    via `useSearchParams` from `react-router-dom`, so filtered views are
    shareable/bookmarkable and survive a refresh.
  - Re-fetch `GET /api/activities` whenever the query string changes.
- Show an `is_indoor` badge ("Indoor"/"Outdoor") and duration (when set) on
  each activity card and on the detail page from task 02.
- i18n: add `filters.indoor`, `filters.outdoor`, `filters.any`,
  `filters.duration.*` bucket labels to all four locale files.

## Out of scope

- Age, location, weather filters (tasks 04–07) — this task only wires up
  indoor/outdoor and duration.
- Persisting a user's preferred filters across sessions.

## Acceptance criteria

- `GET /api/activities?indoor=true` returns only indoor activities;
  `?indoor=false` only outdoor; omitted returns all.
- `GET /api/activities?max_duration_minutes=30` excludes activities with
  `duration_minutes` null or `> 30`.
- Changing the filter bar updates the URL and the list without a full page
  reload; reloading the page with a filtered URL reproduces the same
  filtered list.
- `make check` passes; `cd frontend && npm run build` succeeds.
