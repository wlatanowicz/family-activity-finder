---
status: todo
---

# 07. Weather-suitability filter and rainy-day shortcut

## Summary

Add a weather-suitability tag to each activity and a filter for it,
completing the discovery summary's MVP filter set ("weather"). Ship a
one-tap "Rainy day?" shortcut on the browse page — pulled forward from the
"Future Features" list because it turns out to be nothing more than this
filter with a preset value, not a separate system (no real weather-API
integration required for MVP).

## Why

Parents most often reach for a weather filter reactively — "it's raining,
what can we do today" — so the UI framing (a shortcut button) matters as
much as the underlying field.

## Backend changes

- `backend/src/apps/activities/models.py` — add:
  - `WeatherSuitability` `StrEnum`: `any`, `sunny_only`, `rainy_friendly`.
    Use the existing `to_sql_enum` helper (`src/utils/db.py`) the same way
    `UserStatus`/`AuthProvider` do, since this is a small, stable, closed
    set (unlike `category` in task 01).
  - `Activity.weather_suitability: WeatherSuitability` — not nullable,
    default `WeatherSuitability.any`.
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional query param `weather: Literal["sunny", "rainy"] | None`:
  - `weather=rainy` → include rows where `weather_suitability` is `any` or
    `rainy_friendly`.
  - `weather=sunny` → include rows where `weather_suitability` is `any` or
    `sunny_only`.
  - Omitted → no filtering (current behavior). Combines with `AND`
    alongside all filters from tasks 03–06.
- Extend `test_routes.py` with cases for each `weather` value and the
  default (`any`) always matching.

## Data changes

- `make make-migrations` — new enum type + column,
  `server_default='any'` so existing rows backfill cleanly.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx`: add a small row of quick-filter chips above/alongside
  the existing filter bar: "☀️ Sunny day", "🌧️ Rainy day", and a way to
  clear back to "Any weather" — wired into the same `useSearchParams`
  filter state (`?weather=rainy`).
  - The "🌧️ Rainy day" chip is the "rainy-day suggestions" feature: no
    separate page or logic, just this filter value plus (for extra
    visibility) auto-combining it with `indoor=true` from task 03 when
    the user taps it specifically labeled as a shortcut — i.e. tapping
    "Rainy day" sets both `weather=rainy` and `indoor=true` in one click,
    since an indoor activity is what "rainy day" actually means to a
    parent. Sunny-day chip does not force `indoor=false` (outdoor-only
    would be too restrictive on a nice day).
- Show a small weather-suitability icon/badge on cards where
  `weather_suitability != any`.
- i18n: `filters.weather.sunny`, `.rainy`, `.any` — all four locale files.

## Out of scope

- Real weather-API integration (auto-detecting today's actual weather) —
  explicitly deferred; this task is a manually-selected filter only.
- Hourly/forecast-based suggestions.

## Acceptance criteria

- `GET /api/activities?weather=rainy` includes `any` and `rainy_friendly`
  rows, excludes `sunny_only`.
- Tapping the "Rainy day" chip sets both weather and indoor filters and the
  URL reflects both; the list narrows accordingly.
- `make check` passes; `cd frontend && npm run build` succeeds.
