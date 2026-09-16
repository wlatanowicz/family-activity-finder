---
status: todo
---

# 02. Client-side routing and activity detail page

## Summary

Introduce client-side routing (the app currently has none — it's a single
`App.tsx` view) and use it to ship the first navigable page: an activity
detail view reachable by clicking an activity on the browse page, at a
shareable URL.

## Why

Every task from here on (favorites page, ratings on a specific activity,
children management, admin forms) needs more than one screen. Routing is
infrastructure with no user value on its own, so it ships bundled with the
first page that actually needs it rather than as a standalone task.

## Backend changes

- Add `GET /api/activities/{activity_id}` to
  `backend/src/apps/activities/routes.py`:
  - `activity_id: UUID` path param.
  - 404 with `ApiErrorCode.activity_not_found` (added in task 01) when no
    row matches.
  - 503 with `ApiErrorCode.database_not_configured` when `session is None`
    (same convention as the list endpoint and `users` routes).
  - Response: full `Activity` fields (`id`, `name`, `description`,
    `address`, `category`, `created_at`).
- Add a test in a new `backend/src/apps/activities/tests/test_routes.py`
  covering: 404 for unknown id, 200 with the right shape for an existing
  row, and 503 without a database.

## Frontend changes

- Add `react-router-dom` (`^7`) to `frontend/package.json`.
- Wrap the app in a `BrowserRouter` in `frontend/src/main.tsx`.
- Split `App.tsx`'s browse markup into a `pages/BrowsePage.tsx` (list +
  fetch from task 01) and add `pages/ActivityDetailPage.tsx`:
  - Route `/` → `BrowsePage`, `/activities/:activityId` → `ActivityDetailPage`.
  - Keep the header (title, language selector, auth panel/status) in a
    shared `Layout` component rendered around the routed pages via
    `<Outlet />`, so sign-in state persists across navigation.
  - `ActivityDetailPage` fetches `GET /api/activities/:activityId`, shows
    name, category, address, full description; shows a "not found" message
    (translated) on 404; each `BrowsePage` card links to
    `/activities/${id}` with a router `Link`.
- i18n: add `activities.detail.notFound`, `activities.detail.backToBrowse`
  to `en.json` and the other three locale files.

## Data changes

None — reuses the `activities` table from task 01.

## Out of scope

- Deep-linkable filters (query-string state) — introduced incrementally as
  filters are added in tasks 03–07.
- Favorites/ratings on the detail page (tasks 08–09 add to this page).

## Acceptance criteria

- Navigating to `/activities/<real-id>` directly (e.g. page refresh, pasted
  link) renders the detail page without going through the browse page.
- Navigating to `/activities/<unknown-id>` shows a translated "not found"
  state, not a crash.
- Clicking an activity card on the browse page navigates without a full
  page reload.
- `make check` passes; `cd frontend && npm run build` succeeds.
