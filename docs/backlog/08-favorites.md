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

---

## Tłumaczenie (PL)

### 08. Ulubione

#### Podsumowanie

Pozwól zalogowanym rodzicom zapisywać aktywności na osobistej liście
ulubionych i przeglądać tę listę na dedykowanej stronie. To pozycja
„Ulubione” z zakresu MVP w podsumowaniu odkrycia.

#### Dlaczego

Ulubione to najprostsza funkcja angażująca, jaką umożliwia autoryzacja już
przygotowana przez szablon, i nie jest wymaganiem wstępnym dla niczego
innego — dobry kolejny krok po tym, jak istnieje już przeglądanie/filtry.

#### Zmiany w backendzie

- **Refaktoryzacja wstępna:** endpoint `me()` w
  `backend/src/apps/users/routes.py` obecnie wbudowuje dekodowanie tokena
  bearer + wyszukiwanie użytkownika. Wydziel to do współdzielonej
  zależności, tak aby to i kolejne zadania (09, 10) nie duplikowały
  logiki:
  - Dodaj `backend/src/apps/users/deps.py` z `get_current_user(creds:
    HTTPAuthorizationCredentials | None = Depends(bearer), session:
    Session | None = Depends(get_db_session)) -> User`, zawierającym
    dokładnie te same sprawdzenia, co obecnie w `me()` (503 przy braku
    sesji, 401 przy braku/błędnych poświadczeniach, 401 przy błędzie
    `decode_token` poprzez `jwt.PyJWTError`, następnie wczytanie wiersza
    `User` — wykorzystaj te same sprawdzenia wyszukiwania/
    `user_not_found`/statusu nieaktywnego, jakie już wykonuje `me()`).
  - Zaktualizuj `me()`, aby korzystał z `Depends(get_current_user)`
    zamiast wbudowanej logiki; potwierdź, że
    `backend/src/apps/users/tests/test_routes.py` nadal przechodzi bez
    zmian (zachowanie nie może się zmienić, tylko lokalizacja kodu).
- Nowa aplikacja `backend/src/apps/favorites/`:
  - `models.py` — tabela `Favorite` o nazwie `favorites`: `id: UUID` pk,
    `user_id: UUID` (`foreign_key="users.id"`, indeksowane),
    `activity_id: UUID` (`foreign_key="activities.id"`, indeksowane),
    `created_at: datetime`. `UniqueConstraint("user_id", "activity_id")`
    — zapisanie dwa razy to operacja bez efektu, a nie zduplikowany
    wiersz.
  - `api_errors.py` — `ApiErrorCode.activity_not_found` (wykorzystaj
    wartość kodu z aplikacji activities lub zdefiniuj lokalnie ten sam
    ciąg znaków — zachowaj spójność dla `translateApiError` we
    frontendzie), `already_favorited` NIE jest błędem (patrz niżej — to
    operacja idempotentna).
  - `routes.py` — `APIRouter(prefix="/api/favorites", tags=["favorites"])`,
    wszystkie trasy wymagają `Depends(get_current_user)`:
    - `GET /api/favorites` — lista ulubionych aktywności bieżącego
      użytkownika, połączona z `Activity`, w tym samym skróconym
      kształcie co `GET /api/activities`.
    - `POST /api/favorites` — treść `{"activity_id": UUID}`; 404
      `activity_not_found`, jeśli aktywność nie istnieje; w przeciwnym
      razie wstaw-lub-zignoruj (idempotentne — wywołanie dwa razy dla tej
      samej aktywności zwraca 200/201 za każdym razem, a nie błąd
      konfliktu).
    - `DELETE /api/favorites/{activity_id}` — usuwa ulubione, jeśli
      istnieje; 200/204 nawet jeśli nie było ulubione (idempotentne
      usuwanie).
  - `__init__.py`
- Zarejestruj `favorites_router` w `backend/src/main.py`.
- Testy w `backend/src/apps/favorites/tests/test_routes.py`: 401 bez
  tokena, 404 dla nieznanej aktywności, idempotentne dodawanie/usuwanie,
  zakres listy (ulubione jednego użytkownika nie przeciekają do `GET`
  innego).

#### Zmiany w danych

- `make make-migrations` — nowa tabela `favorites` z powyższym
  ograniczeniem FK/unikalności.
- `make migrate`.

#### Zmiany we frontendzie

- `frontend/src/auth/api.ts` (lub nowy `favorites/api.ts`): cienkie
  wrappery fetch dla trzech endpointów, dołączające przechowywany token
  bearer w taki sam sposób jak `loadMe`.
- Karta aktywności (`BrowsePage.tsx`) i `ActivityDetailPage.tsx`: przycisk
  z ikoną serca/zapisu.
  - Niezalogowany: przycisk jest nadal widoczny, ale kliknięcie pokazuje
    przetłumaczony monit o zalogowanie (nie ukrywaj funkcji — tak
    użytkownik dowiaduje się, że istnieje).
  - Zalogowany: przełącza stan ulubionego optymistycznie, wywołuje
    `POST`/`DELETE`, synchronizuje w razie błędu.
- Nowa `pages/FavoritesPage.tsx` pod trasą `/favorites` (rozszerza
  routing z zadania 02): pobiera `GET /api/favorites`, renderuje ten sam
  komponent karty co strona przeglądania; stan pusty zachęca do
  przeglądania i zapisywania aktywności. Dodaj link nawigacyjny w
  wspólnym nagłówku `Layout`, widoczny tylko po zalogowaniu.
- i18n: `favorites.title`, `.empty`, `.signInToSave`, `.save`, `.saved`
  — wszystkie cztery pliki lokalizacji.

#### Poza zakresem

- Ulubione per dziecko (np. „zapisz specjalnie dla Mii”) — ulubione są
  przypisane do konta w MVP; profile dzieci (zadanie 10) nie łączą się z
  ulubionymi.
- Powiadomienia/przypomnienia o ulubionych aktywnościach.

#### Kryteria akceptacji

- Niezalogowani użytkownicy widzą przycisk zapisu, ale są proszeni o
  zalogowanie, a nie po cichu blokowani lub pozbawieni tej opcji.
- Zapisanie tej samej aktywności dwukrotnie nie tworzy dwóch wierszy ani
  nie zwraca błędu.
- `/favorites` pokazuje dokładnie zapisane aktywności bieżącego
  użytkownika i nic z innych kont.
- Zachowanie `me()` jest niezmienione po wydzieleniu zależności
  (istniejące testy autoryzacji nadal przechodzą bez modyfikacji).
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
