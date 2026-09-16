---
status: todo
---

# 16. Anonymized child context on reviews

## Summary

Let a parent optionally attach one of their saved child profiles (task 10)
to a rating/review (task 09) when submitting it, and show that context to
other readers in fully anonymized form (e.g. "Parent of a 4-year-old"
instead of any name or identity). This closes the remaining half of the
"profil dziecka... uzywany przy filtrowaniu i dodawaniu opinii... uzywamy
zaononimizowanych danych przy opiniach" idea from `docs/backlog/IDEAS.md` —
the filtering half already shipped as task 11; the review-linking half
never had a task.

## Why

`IDEAS.md` describes the child profile as feeding two things: filtering
(task 11) and reviews, with an explicit privacy constraint that the profile
itself stays private and only anonymized data (the child's age, never name
or identity) surfaces in reviews. Today task 09's reviews only show a
masked identity for the *account*, with no link to *which child* the review
is actually about — a detail other parents reading a review would find
useful ("this was written by a parent of a 4-year-old" is more relevant
context than a masked email).

## Backend changes

- `backend/src/apps/ratings/models.py` — add to `Rating`:
  - `child_id: UUID | None` — `foreign_key="child_profiles.id"`, nullable
    (attaching a child is optional; a rating with none behaves exactly as
    task 09 already specifies).
- `backend/src/apps/ratings/routes.py`:
  - `POST /api/activities/{activity_id}/ratings` (task 09): extend the
    request body with optional `child_id: UUID | None`. If supplied,
    verify it belongs to the calling user (404 `child_not_found` if not —
    same non-leaking pattern task 10 uses) and store it on the rating.
  - `GET /api/activities/{activity_id}/ratings` (task 09): when a rating
    has a `child_id`, compute the linked child's current `age_years` (same
    computation task 10's `GET /api/children` uses) server-side and
    include it as `child_age_years: int | None` in the per-rating response
    — **never** include the child's name, id, or the owning user's
    identity beyond what task 09 already exposes. This is the anonymized
    data `IDEAS.md` calls for: an age number, nothing that could identify
    the specific child or account.
- Tests: rating with a `child_id` returns the correct `child_age_years` in
  the public listing without leaking the child's name/id; a `child_id`
  belonging to another user is rejected (404, not silently accepted or
  ignored); rating with no `child_id` behaves exactly as before this task.

## Data changes

- `make make-migrations` — new nullable `child_id` FK column on `ratings`.
- `make migrate`.

## Frontend changes

- `ActivityDetailPage.tsx`'s rating submission widget (task 09): when
  signed in with at least one saved child (task 10), add an optional
  "Reviewing for" select listing the user's children by name (name is only
  ever shown to the reviewer themselves, filling out their own form — never
  sent anywhere but this request) with a "Prefer not to say" default;
  submitting includes the chosen `child_id`.
- Reviews list: render `child_age_years` when present as a small
  translated line under the review, e.g. "Parent of a 4-year-old" — never
  render a child's name or any other identifying detail, since none is
  ever returned by the API for this purpose.
- i18n: `ratings.reviewingFor.label`, `.notSaid`,
  `ratings.childContext` (`"Parent of a {{age}}-year-old"`) — all four
  locale files.

## Out of scope

- Attaching more than one child to a single rating — one optional child
  per rating, matching the "For Mia" single-child selection pattern task
  11 already established.
- Any use of this link for recommendation/personalization beyond display —
  that would extend task 11's scope, not this one.

## Acceptance criteria

- A rating submitted with a `child_id` shows the child's current age (not
  name, not id) on the public reviews list; a rating submitted without one
  shows no child context line, matching task 09's current behavior.
- A user cannot attach another user's child to their rating (404, no data
  leak).
- Nothing in the public API response for ratings ever includes a child's
  name or id — only the derived age.
- `make check` passes; `cd frontend && npm run build` succeeds.
