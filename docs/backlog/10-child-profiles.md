---
status: todo
---

# 10. Child profiles

## Summary

Let signed-in parents save one or more children's profiles (name + birth
date) to their account, with a "My Children" management page. This doesn't
change the browse experience yet — that wiring is task 11 — it's purely the
data entry/management screen.

## Why

Manually typing an age into the filter (task 04) works, but the product's
actual differentiator per the discovery summary ("a decision engine, not
another directory") is personalization — knowing a parent has a 4-year-old
without them re-entering it every visit. This task lays that data down;
task 11 spends it.

## Backend changes

- New app `backend/src/apps/children/`:
  - `models.py` — `ChildProfile` table `child_profiles`: `id: UUID` pk,
    `user_id: UUID` (`foreign_key="users.id"`, indexed), `name: str`
    (`max_length=100`), `birth_date: date`, `created_at: datetime`.
  - `api_errors.py` — `ApiErrorCode.child_not_found`.
  - `routes.py` — `APIRouter(prefix="/api/children", tags=["children"])`,
    all routes require `Depends(get_current_user)` (from task 08's
    `users/deps.py`):
    - `GET /api/children` — list the current user's children, each with a
      derived `age_years` computed server-side from `birth_date` (so the
      frontend never re-implements age math — reused again in task 11).
    - `POST /api/children` — body `{"name": str, "birth_date": date}`.
      Reject future `birth_date` (422) and implausible ages (e.g.
      `birth_date` more than 21 years ago — this product's scope is
      children's activities; a soft validation ceiling avoids garbage
      input, not a hard product rule).
    - `PATCH /api/children/{child_id}` — partial update, scoped to the
      caller; 404 (`child_not_found`) if the id doesn't belong to them
      (don't leak existence of other users' rows with a 403 vs 404
      distinction).
    - `DELETE /api/children/{child_id}` — scoped to the caller, idempotent.
  - `__init__.py`
- Register `children_router` in `backend/src/main.py`.
- Tests in `backend/src/apps/children/tests/test_routes.py`: CRUD scoped
  to the caller (one user can't read/edit/delete another's child), age
  computation correctness, validation errors for future/implausible
  birth dates.

## Data changes

- `make make-migrations` — new `child_profiles` table.
- `make migrate`.

## Frontend changes

- Add `@mantine/dates` and `dayjs` to `frontend/package.json` (Mantine's
  date input needs both; not currently installed).
- New `pages/ChildrenPage.tsx` at route `/children` (routing from task 02):
  - List of the signed-in user's children as cards (name, computed age).
  - "Add child" form: name text input + `DateInput` for birth date.
  - Edit/delete actions per child.
  - Route is only reachable/shown in nav when signed in; redirect (or show
    a sign-in prompt) if a signed-out user hits `/children` directly.
- Nav link "My Children" in the shared `Layout` header, shown when signed
  in (alongside the "Favorites" link added in task 08).
- i18n: `children.title`, `.addChild`, `.name`, `.birthDate`, `.age`,
  `.empty`, `.deleteConfirm` — all four locale files.

## Out of scope

- Wiring child profiles into activity filtering/recommendations — that's
  task 11.
- Linking a child profile to a submitted review for anonymized context —
  that's task 16.
- Photos or additional child attributes (interests, etc.) beyond name and
  birth date — kept minimal for MVP.

## Acceptance criteria

- A user can add, edit, and remove children; another signed-in user's
  children never appear in their list or are editable by them.
- `age_years` returned by the API matches the birth date (e.g. a birth
  date exactly 4 years ago today reports `4`).
- Signed-out access to `/children` does not error the app — it prompts to
  sign in.
- `make check` passes; `cd frontend && npm run build` succeeds.
