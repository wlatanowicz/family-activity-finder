---
status: todo
---

# 20. Mobile child profiles and personalized recommendations

## Summary

Port child profile management (task 10) and the "for my child" one-tap
age-filter shortcuts (task 11) to the `mobile/` app.

## Why

This closes out mobile parity with the web MVP feature set: the
personalization loop (save a child once, get one-tap filtered results and
anonymized-review context everywhere else) should work the same regardless
of which client a parent used to set it up.

## Backend changes

None — reuses `GET/POST/PATCH/DELETE /api/children` (task 10) and the
`child_id` param on `GET /api/activities` (task 11) exactly as already
defined.

## Data changes

None.

## Frontend changes

- New `ChildrenScreen`: list the signed-in user's children (name, computed
  age), add/edit/delete, using `@react-native-community/datetimepicker`
  (the native equivalent of the web's `@mantine/dates` `DateInput`) for
  birth date entry.
- Browse screen (task 18): render a row of chips — one per saved child,
  e.g. "For Mia (4y)" — above the filter controls when signed in with at
  least one saved child; tapping one sets the `child_id` query param the
  same way the web `BrowsePage.tsx` does, with a clear affordance to
  return to the manual age filter.
- A child saved/edited on mobile must be immediately usable for filtering
  on web (and vice versa) since both hit the same `/api/children` data —
  no client-side caching that could go stale between the two apps beyond
  a normal refetch-on-focus.
- i18n: reuse/extend the keys already defined for tasks 10/11 on web.

## Out of scope

- Any change to the recommendation logic itself (still a plain age-range
  match, per task 11's own out-of-scope note) — this task only ports the
  existing behavior to a new client.

## Acceptance criteria

- A child added on mobile appears correctly in the web app's "My
  Children" page and its age-filter chip, and vice versa.
- Selecting a child chip on mobile produces the same filtered result set
  as manually entering that child's current age.
- A user cannot access another user's children from the mobile app (same
  404/scoping behavior as web, since it's the same endpoint).
