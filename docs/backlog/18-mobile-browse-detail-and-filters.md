---
status: todo
---

# 18. Mobile browse, detail, and filters

## Summary

Port the web browse page, activity detail page, and every filter (indoor/
outdoor, duration, age, city, "near me," weather) to native screens in the
`mobile/` app from task 17, against the exact same `GET /api/activities`
endpoint and query parameters tasks 01–07 already define.

## Why

Browse + filter is the core "help parents discover activities" experience
the whole product is built around (tasks 01–07); it has to exist on mobile
for the platform to be more than an auth shell, and it needs no new backend
work since the API is already client-agnostic.

## Backend changes

None — reuses `GET /api/activities` and `GET /api/activities/{id}` exactly
as tasks 01–07 defined them for the web frontend.

## Data changes

None.

## Frontend changes

- New `BrowseScreen` in `mobile/`: fetches `GET /api/activities`, renders a
  scrollable list of activity cards (name, category badge, address,
  truncated description — same fields as the web `BrowsePage.tsx`).
- New `ActivityDetailScreen`, navigated to on card tap, fetches
  `GET /api/activities/{id}` (task 02's endpoint).
- Filter UI native equivalents for every task 03–07 filter, all
  contributing to the same query params the web app builds
  (`is_indoor`, `max_duration_minutes`, `age_years`, `city`, `lat`/`lng`,
  weather tag): a filter sheet/modal is a more natural native pattern than
  the web's inline filter bar — use whichever native pattern (bottom
  sheet, dedicated filter screen) fits the chosen navigation library from
  task 17, as long as the resulting query params match.
- "Near me" (task 06): use `expo-location`'s permission request +
  `getCurrentPositionAsync` in place of the browser Geolocation API;
  same opt-in-only rule — never request location on screen mount, only on
  an explicit "Use my location" tap.
- Rainy-day shortcut (task 07): a one-tap button applying the weather
  filter's preset value, same as the web version.
- i18n: reuse/extend the same translation keys ported in task 17.

## Out of scope

- The map view from task 13 — `react-native-maps` needs native module
  linking and an EAS development build rather than Expo Go, which is a
  meaningfully bigger lift than the list-based screens here. Track it as
  its own follow-up task if a mobile map is wanted later; this task ships
  list-view browsing only.
- Favorites, ratings/reviews, child profiles — tasks 19–20.

## Acceptance criteria

- The mobile browse screen and the web browse page return/display the
  same activities for equivalent filter selections (same query params
  against the same backend).
- Tapping an activity opens its detail screen with the same data the web
  detail page shows.
- The "near me" filter correctly handles a denied location permission
  without crashing the screen (mirrors task 06's web acceptance criterion).
- Every filter from tasks 03–07 is reachable and combinable on mobile.
