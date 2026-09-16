---
status: todo
---

# 06. "Near me" geolocation distance sort

## Summary

Add coordinates to activities and let parents sort the browse list by
distance from their current location, using the browser's Geolocation API.
This is the second, richer slice of the discovery summary's "location-aware
search," building on the city filter from task 05.

## Why

"What's closest to me right now" is a distinct use case from "what's in
this city" (task 05) — e.g. a parent near a city border, or comparing two
close-by options within the same city. Coordinates are also needed
groundwork for any future map view.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `latitude: float | None`
  - `longitude: float | None`
  - Both nullable — coordinates get backfilled as activities are
    created/edited (task 12); rows without them simply can't be
    distance-sorted.
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional `lat: float | None`, `lng: float | None` (must be provided
  together — 422 if only one is set):
  - When both are provided, compute the haversine distance in Python
    between `(lat, lng)` and each row's `(latitude, longitude)`, attach it
    as `distance_km` in the response, and sort ascending by distance.
    Rows with null coordinates get `distance_km: null` and sort last (not
    excluded — a parent should still see them, just below located results).
  - Combines with all existing filters (`indoor`, `max_duration_minutes`,
    `age_years`, `city`) via `AND`; sorting by distance only kicks in when
    `lat`/`lng` are present, otherwise sort order is unchanged (current
    default, e.g. by name or `created_at`).
  - Put the haversine helper in `backend/src/apps/activities/geo.py` so
    it's unit-testable in isolation.
- Add `backend/src/apps/activities/tests/test_geo.py` (known-distance pairs,
  e.g. two coordinates ~1km apart) and extend `test_routes.py` for the
  sorting/null-handling behavior and the "only one of lat/lng" 422 case.

## Data changes

- `make make-migrations` — two new nullable float columns.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx`: add a "Use my location" button. On click, call
  `navigator.geolocation.getCurrentPosition`; on success, store
  `{lat, lng}` in component state and add them to the `useSearchParams`
  filter state (`?lat=...&lng=...`) so results resort; on denial/error,
  show a translated inline message and leave sorting unchanged (don't
  block the rest of the page).
  - This is a real device permission prompt — keep it strictly
    opt-in/behind the button; never request location automatically on
    page load.
- Show `distance_km` (formatted, e.g. "2.3 km away") on each card when
  present, once location is active.
- i18n: `filters.nearMe.button`, `filters.nearMe.denied`,
  `filters.nearMe.unsupported`, `activities.distanceAway` — all four locale
  files.

## Out of scope

- A map view — this task only adds list sorting, no visual map (see task
  13).
- Editing coordinates from the UI (task 12 handles data entry).
- Configurable radius cutoff (e.g. "within 10 km") — sort-only for MVP.

## Acceptance criteria

- With `lat`/`lng` supplied, activities with coordinates are returned
  nearest-first with a correct `distance_km`; activities without
  coordinates appear after all located ones.
- Supplying only `lat` (no `lng`) returns 422.
- Denying the browser location permission leaves the browse page usable
  (no crash, no blocked UI), with a translated explanation shown.
- `make check` passes; `cd frontend && npm run build` succeeds.
