---
status: todo
---

# 08. Favorites

## Summary

Let signed-in parents save activities to a personal favorites list, and
view that list on its own page. This is the "Favorites" line item from the
discovery summary's MVP scope.

## Why

Favorites is the simplest engagement feature the auth already scaffolded by
the template enables, and it's a prerequisite for nothing else — good next
step after the browse/filter experience is in place.

## Backend changes

- **Prerequisite refactor:** `backend/src/apps/users/routes.py`'s `me()`
  endpoint currently inlines bearer-token decoding + user lookup. Extract
  that into a reusable dependency so this and later tasks (09, 10) don't
  duplicate it:
  - Add `backend/src/apps/users/deps.py` with `get_current_user(creds:
    HTTPAuthorizationCredentials | None = Depends(bearer), session: Session
    | None = Depends(get_db_session)) -> User`, containing the exact
    checks currently in `me()` (503 if no session, 401 if no/bad creds, 401
    on `decode_token` failure via `jwt.PyJWTError`, then load the `User` row
    — reuse whatever lookup/`user_not_found`/inactive-status checks `me()`
    already does).
  - Update `me()` to use `Depends(get_current_user)` instead of its inline
    logic; confirm `backend/src/apps/users/tests/test_routes.py` still
    passes unmodified (behavior must not change, only where the code
    lives).
- New app `backend/src/apps/favorites/`:
  - `models.py` — `Favorite` table `favorites`: `id: UUID` pk,
    `user_id: UUID` (`foreign_key="users.id"`, indexed),
    `activity_id: UUID` (`foreign_key="activities.id"`, indexed),
    `created_at: datetime`. `UniqueConstraint("user_id", "activity_id")` —
    saving twice is a no-op, not a duplicate row.
  - `api_errors.py` — `ApiErrorCode.activity_not_found` (reuse the
    activities app's code value or define locally with the same string —
    keep it consistent for the frontend's `translateApiError`),
    `already_favorited` is NOT an error (see below — this is idempotent).
  - `routes.py` — `APIRouter(prefix="/api/favorites", tags=["favorites"])`,
    all routes require `Depends(get_current_user)`:
    - `GET /api/favorites` — list the current user's favorited activities,
      joined to `Activity`, same summary shape as `GET /api/activities`.
    - `POST /api/favorites` — body `{"activity_id": UUID}`; 404
      `activity_not_found` if the activity doesn't exist; otherwise
      insert-or-ignore (idempotent — calling it twice for the same
      activity returns 200/201 both times, not a conflict error).
    - `DELETE /api/favorites/{activity_id}` — removes the favorite if
      present; 200/204 even if it wasn't favorited (idempotent delete).
  - `__init__.py`
- Register `favorites_router` in `backend/src/main.py`.
- Tests in `backend/src/apps/favorites/tests/test_routes.py`: 401 without a
  token, 404 for unknown activity, idempotent add/remove, list scoping (one
  user's favorites don't leak into another's `GET`).

## Data changes

- `make make-migrations` — new `favorites` table with the FK/unique
  constraint above.
- `make migrate`.

## Frontend changes

- `frontend/src/auth/api.ts` (or a new `favorites/api.ts`): thin fetch
  wrappers for the three endpoints, attaching the stored bearer token the
  same way `loadMe` does.
- Activity card (`BrowsePage.tsx`) and `ActivityDetailPage.tsx`: a
  heart/save icon button.
  - Signed out: button is still visible but clicking it shows a translated
    prompt to sign in (don't hide the feature — that's how a user discovers
    it exists).
  - Signed in: toggles favorited state optimistically, calls
    `POST`/`DELETE`, reconciles on failure.
- New `pages/FavoritesPage.tsx` at route `/favorites` (extends the routing
  from task 02): fetches `GET /api/favorites`, renders the same card
  component as the browse page; empty state prompts to browse and save
  some activities. Add a nav link to it in the shared `Layout` header, only
  shown when signed in.
- i18n: `favorites.title`, `.empty`, `.signInToSave`, `.save`, `.saved` —
  all four locale files.

## Out of scope

- Favoriting per-child (e.g. "save for Mia specifically") — favorites are
  per-account for MVP; child profiles (task 10) don't attach to favorites.
- Notifications/reminders about favorited activities.

## Acceptance criteria

- Signed-out users see the save button but are prompted to sign in, not
  silently blocked or hidden.
- Saving the same activity twice does not create two rows or error.
- `/favorites` shows exactly the current user's saved activities and
  nothing from other accounts.
- `me()` behavior is unchanged after the dependency extraction (existing
  auth tests still pass without modification).
- `make check` passes; `cd frontend && npm run build` succeeds.
