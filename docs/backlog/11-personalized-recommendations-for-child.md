---
status: todo
---

# 11. Personalized "for my child" recommendations

## Summary

Connect child profiles (task 10) to the age filter (task 04): signed-in
parents with saved children see one-tap shortcuts on the browse page — "For
Mia (4y)" — that apply the age filter without typing a number.

## Why

This is the smallest possible slice of "personalized suggestions" from the
discovery summary's Future Features list that's cheap enough to pull into
MVP: it's a convenience wrapper around two features that already exist
(tasks 04 and 10), not a new recommendation system.

## Backend changes

- `backend/src/apps/activities/routes.py`: extend `GET /api/activities`
  with an optional `child_id: UUID | None` query param, accepted alongside
  the existing `age_years` param (mutually convenient, not exclusive —
  `child_id` is just a server-side shortcut that resolves to an age):
  - Auth is required only when `child_id` is supplied (the endpoint stays
    public otherwise) — use `Depends(get_current_user)` conditionally, or
    simplest: accept an optional bearer token and 401 only if `child_id`
    is present but the token is missing/invalid.
  - Resolve `child_id` to the child's current age (same computation as
    task 10's `GET /api/children`), verify it belongs to the calling user
    (404 `child_not_found` if not — do not allow looking up another
    user's child's age via this param), and apply the same filtering logic
    `age_years` already uses (task 04). If both `age_years` and `child_id`
    are supplied, `child_id` wins (it's the more specific signal).
- Extend `test_routes.py`: `child_id` resolves to the right age filter,
  404 for a child that doesn't belong to the caller, 401 when `child_id`
  is supplied without auth.

## Data changes

None — reuses `activities` (task 01/04) and `child_profiles` (task 10).

## Frontend changes

- `BrowsePage.tsx`: when signed in and the user has at least one saved
  child (fetch `GET /api/children` once on mount, same as `ChildrenPage`),
  render a row of chips above the filter bar: one per child, e.g. "For Mia
  (4y)". Tapping one sets `?childId=<id>` in the `useSearchParams` filter
  state (replacing any manually-set `age`), and a "clear" affordance
  returns to the unfiltered/general age input from task 04.
  - Signed-out users or users with no saved children see the existing
    manual age input from task 04 with no change — this task is additive,
    not a replacement.
- i18n: `filters.forChild` (`"For {{name}} ({{age}}y)"`), `.clear` — all
  four locale files.

## Out of scope

- Any ranking/scoring beyond the existing age-range match (that's the
  "AI recommendations" future feature, intentionally deferred — see
  `docs/backlog/README.md`).
- Combining multiple children into one filter (e.g. "activities good for
  both my kids") — one child selected at a time for MVP.

## Acceptance criteria

- Selecting a child chip produces the same filtered result set as manually
  entering that child's exact current age into the task 04 input.
- A user cannot use another user's `child_id` to filter (404, not leaked
  age data).
- The chips only render for signed-in users with at least one saved child;
  everyone else sees the unchanged manual age input.
- `make check` passes; `cd frontend && npm run build` succeeds.
