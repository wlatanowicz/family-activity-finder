---
status: todo
---

# 15. Review photo attachments

## Summary

Let parents attach up to a few photos to a rating/review submitted in task
09. This is the remaining half of the "ocenianie z mozliwoscia dodowania
opisu i zdjec" (rating with the ability to add a description and photos)
idea from `docs/backlog/IDEAS.md` — the description half (the `comment`
field) already shipped as part of task 09, which explicitly listed photo
attachments as out of scope.

## Why

`IDEAS.md` names photos as part of the same idea as the written review, not
a separate one — real-world photos of an activity (a park, a play area, a
class in session) are strong trust signals for other parents deciding
whether to go, on top of the star score and text.

## Backend changes

- Add object storage for uploaded images (reuse whatever storage backend
  the template/infra already provisions for file uploads, e.g. an S3-
  compatible bucket referenced in `backend/src/config.py`/infra scripts; if
  none exists yet, provision the smallest viable bucket and wire its
  name/region through the existing settings pattern rather than introducing
  a new config mechanism).
- `backend/src/apps/ratings/models.py` — new table `rating_photos`:
  `id: UUID` pk, `rating_id: UUID` (`foreign_key="ratings.id"`, indexed),
  `storage_key: str`, `created_at: datetime`. A rating can have zero or
  more photos (one-to-many, not a column on `ratings`).
- `backend/src/apps/ratings/routes.py`:
  - `POST /api/activities/{activity_id}/ratings/photos` — auth required,
    `multipart/form-data` upload, limited to image content types
    (`image/jpeg`, `image/png`, `image/webp`) and a size cap (e.g. 5 MB);
    422 on anything else. Requires the caller to already have a rating on
    this activity (create the rating first via task 09's endpoint, then
    attach photos to it) — 404 if they don't. Cap the count per rating
    (e.g. 5) — 422 past the cap. Stores the file, inserts a
    `rating_photos` row, returns its id + a servable URL.
  - `DELETE /api/activities/{activity_id}/ratings/photos/{photo_id}` — auth
    required, scoped to the caller's own rating; idempotent.
  - Extend `GET /api/activities/{activity_id}/ratings`'s per-rating shape
    with a `photos: [{"id", "url"}, ...]` array.
  - Extend `DELETE .../ratings/me` (task 09) to also delete any attached
    photo rows and their stored objects, so deleting a rating doesn't leave
    orphaned files.
- Tests: upload succeeds and appears in the `GET` listing, rejects
  non-image content types and oversized files, enforces the per-rating
  photo cap, deleting a rating cascades its photos, a user can't delete
  another user's photo.

## Data changes

- `make make-migrations` — new `rating_photos` table with FK to `ratings`.
- `make migrate`.

## Frontend changes

- `ActivityDetailPage.tsx`'s rating submission widget (task 09): after
  saving a rating, show a photo upload control (file input, multiple
  select up to the backend cap) that calls the new upload endpoint per
  file; show upload progress/errors per file, not a single blocking
  spinner for the whole batch.
- Reviews list: render each review's attached photos as a small thumbnail
  strip; clicking a thumbnail opens it larger (a simple lightbox/modal is
  enough — no gallery library needed for a handful of images).
- Allow the review author to delete their own attached photos individually
  from the same widget.
- i18n: `ratings.photos.add`, `.uploading`, `.tooLarge`, `.wrongType`,
  `.limitReached`, `.remove` — all four locale files.

## Out of scope

- Image moderation (inappropriate content scanning) — flagged as a
  follow-up once real usage surfaces the need, same rationale task 09 gave
  for review moderation generally.
- Client-side image cropping/editing — raw upload only for MVP.
- Photos attached anywhere other than a rating (e.g. directly on an
  `Activity` by an admin) — task 12's admin form is unaffected by this
  task.

## Acceptance criteria

- A signed-in user can attach photos to their own rating and see them in
  the public reviews list immediately.
- Uploading a non-image file or a file over the size cap is rejected with a
  translated error, not a silent failure or a crash.
- Deleting a rating (task 09's own-rating delete) leaves no orphaned photo
  rows or stored files.
- `make check` passes; `cd frontend && npm run build` succeeds.
