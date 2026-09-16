---
status: todo
---

# 09. Community ratings and reviews

## Summary

Let signed-in parents rate (1–5 stars) and optionally review an activity;
show the average rating and review count on the browse list and detail
page. This is the "Community ratings" line item from the discovery
summary's MVP scope, and the direct answer to the "building a trusted
review community" challenge it also names.

## Why

Ratings are the product's core trust signal — the thing that makes it
better than a plain directory. It depends only on auth (already scaffolded)
and the `Activity` model (task 01), so it can ship independently of
favorites (task 08), but comes after it in this backlog since it reuses the
`get_current_user` dependency task 08 extracts.

## Backend changes

- New app `backend/src/apps/ratings/`:
  - `models.py` — `Rating` table `ratings`: `id: UUID` pk,
    `user_id: UUID` (`foreign_key="users.id"`, indexed),
    `activity_id: UUID` (`foreign_key="activities.id"`, indexed),
    `score: int` (1–5, enforce with a Pydantic validator on the request
    model, not a DB constraint, to keep the migration simple),
    `comment: str | None` (`max_length=1000`), `created_at: datetime`,
    `updated_at: datetime`. `UniqueConstraint("user_id", "activity_id")` —
    one rating per user per activity; re-rating updates the existing row
    (upsert) rather than creating a second one.
  - `api_errors.py` — `ApiErrorCode.activity_not_found`, `rating_not_found`.
  - `routes.py`:
    - `POST /api/activities/{activity_id}/ratings` — auth required
      (`Depends(get_current_user)` from task 08's `users/deps.py`). Body
      `{"score": int, "comment": str | None}`. 404 if the activity doesn't
      exist. Creates the user's rating for this activity, or updates it in
      place if one already exists (`updated_at` bumped).
    - `GET /api/activities/{activity_id}/ratings` — public, no auth.
      Query params `limit`/`offset` (default `limit=20, offset=0`).
      Returns `{"average_score": float | None, "count": int, "ratings":
      [{"user_email"-or-similar-display-name, "score", "comment",
      "created_at"}, ...]}`. Decide on a privacy-safe display name (e.g.
      the local part of the email, or "a parent" + first name only if the
      `User` model gains a display name later — for now, do not expose
      full email; truncate/mask it, e.g. `j***@example.com` pattern, or
      simply omit identity and show "Verified parent").
    - `DELETE /api/activities/{activity_id}/ratings/me` — auth required,
      removes the current user's own rating if present (idempotent).
  - `__init__.py`
- `backend/src/apps/activities/routes.py`: extend both
  `GET /api/activities` and `GET /api/activities/{id}` responses with
  `average_rating: float | None` and `ratings_count: int`, computed via a
  SQL aggregate (e.g. a `LEFT JOIN`/subquery against `ratings`) rather than
  denormalized on `Activity` — simplest correct approach for MVP data
  volumes.
- Register `ratings_router` in `backend/src/main.py`.
- Tests in `backend/src/apps/ratings/tests/test_routes.py`: create then
  re-rate the same activity (upsert, still one row), 404 for unknown
  activity, average/count math with multiple users' ratings, own-rating
  delete is scoped to the caller.

## Data changes

- `make make-migrations` — new `ratings` table with FK/unique constraint.
- `make migrate`.

## Frontend changes

- `ActivityDetailPage.tsx`:
  - Average rating + count shown near the title (e.g. "★ 4.3 (12 ratings)").
  - A star-input widget (1–5) + optional comment textarea, shown to signed-in
    users; submitting calls `POST .../ratings`. If the user already rated
    this activity, prefill the widget with their existing score/comment
    (fetch their own rating — either from the `GET .../ratings` list by
    matching current user, or accept a small added convenience: the `POST`
    response returns the saved rating, cache it client-side after first
    submit).
  - Reviews list below, paginated (`limit`/`offset`), each showing score,
    comment, masked identity, relative date.
  - Signed-out users see the reviews list and average but a translated
    prompt to sign in in place of the input widget.
- Browse-page cards (`BrowsePage.tsx`): show the average rating badge (e.g.
  "★ 4.3") when `ratings_count > 0`; omit the badge entirely when there are
  no ratings yet (avoid showing "★ 0" or "no ratings" clutter on every
  card).
- i18n: `ratings.average`, `.count`, `.submit`, `.update`, `.signInToRate`,
  `.commentPlaceholder`, `.empty` — all four locale files.

## Out of scope

- Moderation/reporting abusive reviews — flagged as a follow-up once real
  usage surfaces the need, not speculatively built now.
- Photo attachments on reviews (task 15).
- Linking a review to a saved child profile for anonymized context
  (task 16).

## Acceptance criteria

- Rating the same activity twice as the same user results in one row with
  the latest score/comment, not two.
- `average_rating`/`ratings_count` on `GET /api/activities` match what
  `GET /api/activities/{id}/ratings` reports for the same activity.
- Signed-out users can read ratings but the submit UI shows a sign-in
  prompt instead of a form.
- `make check` passes; `cd frontend && npm run build` succeeds.
