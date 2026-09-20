---
status: todo
---

# 19. Mobile favorites and ratings

## Summary

Port favorites (task 08) and community ratings/reviews — including photo
attachments (task 15) and the optional anonymized child context (task 16)
— to the `mobile/` app, against the same endpoints the web frontend uses.

## Why

These are the engagement/trust-loop features (favorites, ratings, photos,
child context) already scoped and built for web; shipping them on mobile
is what makes the native app a full alternative to the web app rather than
a read-only browse shell.

## Backend changes

None — reuses `POST/DELETE /api/favorites`, `GET/POST/DELETE
/api/activities/{id}/ratings[...]`, the photo upload/delete endpoints from
task 15, and the `child_id`/`child_age_years` fields from task 16 exactly
as already defined for the web frontend.

## Data changes

None.

## Frontend changes

- Favorites (task 08): a save/heart button on activity cards and the
  detail screen, calling the same `POST`/`DELETE /api/favorites`
  endpoints; signed-out tap shows a native prompt to sign in instead of
  hiding the button. A `FavoritesScreen` listing the signed-in user's
  saved activities.
- Ratings/reviews (task 09): star-input + optional comment on the detail
  screen, submitting to the same `POST .../ratings` endpoint; a reviews
  list below showing score, comment, masked identity, relative date, same
  privacy rules as web (no full email ever shown).
- Photo attachments (task 15): use `expo-image-picker` (camera or library)
  to let a user attach up to the backend's per-rating cap, uploading via
  the same multipart endpoint; render each review's photos as a
  thumbnail strip with a full-screen viewer on tap.
- Anonymized child context (task 16): an optional "Reviewing for" picker
  (native equivalent of the web select) listing the user's saved children
  by name — shown only to the reviewer while filling out their own
  review — submitting the chosen `child_id`; reviews list renders
  `child_age_years` when present (e.g. "Parent of a 4-year-old"), never a
  name, exactly as the API already guarantees.
- i18n: reuse/extend the keys already defined for tasks 08/09/15/16 on
  web.

## Out of scope

- Anything not already in scope for tasks 08/09/15/16 on web (e.g.
  moderation, per-child favorites) — this task is a straight port, not a
  chance to expand scope.

## Acceptance criteria

- Favoriting/unfavoriting on mobile is reflected in `GET /api/favorites`
  identically to the web flow (same account, same result set).
- A rating submitted from mobile — with an attached photo and an optional
  linked child — appears correctly in the web app's reviews list (average
  score, photo thumbnail, anonymized child age line), confirming both
  clients share one backend and one data model.
- Signed-out users can read ratings/photos but are prompted to sign in
  before submitting one, matching the web behavior.
