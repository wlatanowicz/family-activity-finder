---
status: todo
---

# 13. Activity map view

## Summary

Add a map view to the browse page showing every filtered activity as a
marker at its coordinates, alongside the existing list view. This is the
"mapa z zaznaczonymi miejscami" idea from `docs/backlog/IDEAS.md` — a
literal map with marked places, not just the distance sort task 06 already
ships.

## Why

Task 06 added coordinates and "near me" sorting but explicitly left the map
itself out of scope ("this task only adds list sorting, no visual map").
Parents scanning "what's around here" often think spatially, and a map is
the most direct answer to that — it's also the only idea in `IDEAS.md` with
no existing task covering it at all.

## Backend changes

None — reuses `GET /api/activities` (including `latitude`/`longitude` from
task 06 and every filter from tasks 03–07); the map is a frontend
presentation of the same data the list view already fetches.

## Data changes

None.

## Frontend changes

- Add a map library to `frontend/package.json` — prefer `react-leaflet` +
  `leaflet` with OpenStreetMap tiles (no API key/billing required, unlike
  Google Maps or Mapbox — keeps this unblocked by any vendor account
  setup).
- `BrowsePage.tsx`: add a list/map view toggle (e.g. segmented control).
  Map view renders one marker per activity that has non-null
  `latitude`/`longitude` from the current filtered result set; activities
  without coordinates are omitted from the map (they still appear in list
  view) with a small translated note ("N activities without a location
  aren't shown on the map").
  - Marker click/tap opens a popup with name, category badge, and a "View
    details" link to the task 02 detail page.
  - Map recenters/refits bounds to the visible markers when filters change.
  - If "Use my location" (task 06) is active, show the user's own position
    as a distinct marker/icon and center the initial view on it; otherwise
    default the initial view to fit all markers (or a neutral world/region
    view when there are none).
- i18n: `browse.viewList`, `.viewMap`, `.mapMissingLocations` — all four
  locale files.

## Out of scope

- Drawing/selecting a search area on the map (e.g. "search this area") —
  filtering stays driven by the existing filter bar and location button,
  not map interaction, for MVP.
- Clustering markers at low zoom — acceptable to skip while activity
  volume is low post-launch; revisit if marker density makes the map
  unreadable.
- A map picker for entering coordinates in the admin form (task 12) —
  already explicitly out of scope there.

## Acceptance criteria

- Switching to map view shows one marker per filtered activity with
  coordinates, and applying a filter (e.g. task 03's indoor/outdoor)
  updates the markers shown.
- An activity with null coordinates never appears on the map but still
  appears in list view for the same filter state.
- Clicking a marker's popup link navigates to that activity's detail page.
- `make check` passes; `cd frontend && npm run build` succeeds.
