---
status: todo
---

# 12. Admin activity management

## Summary

Give the team a way to create, edit, and delete activities through the app
itself, gated to admin accounts — covering every field introduced in tasks
01–07 (name, description, address, category, indoor/outdoor, duration, age
range, city, coordinates, weather suitability).

## Why

This is the direct fix for the discovery summary's named challenge, "cold
start (content)": until now, the only way to populate real activities is a
direct database write. It's ordered last because it needs every activity
column from tasks 01–07 to exist first, so the admin form has real fields
to edit rather than being built twice.

## Backend changes

- `backend/src/apps/users/models.py`: add `is_admin: bool` to `User`, not
  nullable, default `False`.
- `backend/src/apps/users/deps.py` (from task 08): add
  `require_admin(user: User = Depends(get_current_user)) -> User` — 403
  with a new `ApiErrorCode.admin_required` if `user.is_admin` is falsy,
  otherwise returns the user (so admin routes can depend on it directly
  and still get the `User` object).
- `backend/src/apps/activities/routes.py`: add admin-only routes:
  - `POST /api/activities` — body covering every `Activity` field from
    tasks 01–07 (all the nullable ones optional, required ones required);
    `Depends(require_admin)`.
  - `PATCH /api/activities/{activity_id}` — partial update, same field
    set, 404 if unknown id; `Depends(require_admin)`.
  - `DELETE /api/activities/{activity_id}` — hard delete; consider what
    happens to dependent `favorites`/`ratings` rows (task 08/09 add FKs to
    `activities.id`) — cascade-delete them in the same transaction rather
    than leaving orphans or failing on FK constraint. Document this
    cascade explicitly in the route's docstring since it's a destructive,
    hard-to-reverse side effect.
- `backend/src/apps/users/routes.py`'s `UserPublic`/`me()` response:
  include `is_admin` so the frontend knows to show the admin nav link.
- Tests: `require_admin` 403s a non-admin, 200s an admin;
  create/update/delete round-trip on every field; delete cascades
  favorites/ratings for that activity.

## Data changes

- `make make-migrations` — `add_column("users", "is_admin", ...,
  server_default="false")`.
- `make migrate`.
- **Deployment note:** no user is an admin immediately after this
  migration. Bootstrapping the first admin requires one manual step —
  document in this task's PR description (not a UI flow, deliberately: a
  self-serve "become admin" button would defeat the purpose of the gate):
  `UPDATE users SET is_admin = true WHERE email = '<you>';` run once
  against the target database after deploy.

## Frontend changes

- `frontend/src/auth/types.ts`: add `is_admin: boolean` to `MeUser`.
- New `pages/AdminActivitiesPage.tsx` at route `/admin/activities`
  (routing from task 02), rendered only when `currentUser.is_admin` — show
  a translated "not authorized" state (not a raw 403) for any other
  signed-in user who navigates there directly, and redirect signed-out
  users to sign in first.
  - Table/list of all activities with edit and delete actions.
  - A form (create + edit, same component) covering every field: name,
    description, address, category (select from the suggested set in task
    01), indoor/outdoor toggle, duration (number, minutes), age min/max
    (two number inputs), city (text — free entry here, unlike the
    read-only dropdown in task 05, since admins are the ones creating new
    cities), latitude/longitude (two number inputs — plain numeric entry
    is enough for MVP; no map picker), weather suitability (select:
    any/sunny only/rainy-friendly).
  - Delete requires an inline confirm step (not a silent one-click delete),
    given task 09/08's cascade-delete side effect.
- Nav link "Admin" in the shared `Layout` header, shown only when
  `currentUser?.is_admin`.
- i18n: `admin.title`, `.notAuthorized`, `.create`, `.edit`, `.delete`,
  `.deleteConfirm`, plus field labels reusing existing filter/activity
  labels from tasks 01–07 where they already exist (don't duplicate keys
  that already exist for "indoor", "category", etc.) — all four locale
  files.

## Out of scope

- Bulk import (CSV/JSON) — one-at-a-time entry is acceptable to unblock
  cold start; revisit if manual entry proves too slow once real content
  volume is known.
- Roles/permissions beyond a single `is_admin` boolean (e.g. regional
  moderators) — not needed at current scale.
- Map-based coordinate picking — plain lat/lng number inputs are enough
  for MVP; an admin can look up coordinates externally.
- The `suitable_for_gender` field from task 14 — that task ships after this
  one and should extend this form with one more field when it lands,
  rather than this task depending on a not-yet-existing column.

## Acceptance criteria

- A non-admin signed-in user hitting `/admin/activities` sees a translated
  "not authorized" message, not raw JSON or a crash; the corresponding API
  calls return 403, not 404 or 500.
- An admin can create an activity with every field populated and see it
  immediately on the public browse page with all filters (tasks 03–07)
  behaving correctly against it.
- Deleting an activity that has favorites and ratings succeeds and leaves
  no orphaned rows in either table.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 12. Zarządzanie aktywnościami przez administratora

#### Podsumowanie

Daj zespołowi sposób na tworzenie, edytowanie i usuwanie aktywności
bezpośrednio w aplikacji, ograniczony do kont administratorów — obejmujący
każde pole wprowadzone w zadaniach 01–07 (nazwa, opis, adres, kategoria,
wnętrze/na zewnątrz, czas trwania, zakres wieku, miasto, współrzędne,
dopasowanie do pogody).

#### Dlaczego

To bezpośrednie rozwiązanie nazwanego w podsumowaniu odkrycia wyzwania
„zimny start (treść)”: do tej pory jedynym sposobem wprowadzenia
prawdziwych aktywności jest bezpośredni zapis do bazy danych. Zadanie jest
ostatnie w kolejce, ponieważ potrzebuje wszystkich kolumn aktywności z
zadań 01–07, aby formularz administracyjny miał realne pola do edycji,
zamiast być budowanym dwa razy.

#### Zmiany w backendzie

- `backend/src/apps/users/models.py`: dodaj `is_admin: bool` do `User`,
  niepuste, domyślnie `False`.
- `backend/src/apps/users/deps.py` (z zadania 08): dodaj
  `require_admin(user: User = Depends(get_current_user)) -> User` — 403
  z nowym `ApiErrorCode.admin_required`, jeśli `user.is_admin` jest
  fałszywe, w przeciwnym razie zwraca użytkownika (tak aby trasy
  administracyjne mogły zależeć od niej bezpośrednio i nadal otrzymywać
  obiekt `User`).
- `backend/src/apps/activities/routes.py`: dodaj trasy tylko dla
  administratorów:
  - `POST /api/activities` — treść obejmująca każde pole `Activity` z
    zadań 01–07 (wszystkie puste jako opcjonalne, wymagane jako
    wymagane); `Depends(require_admin)`.
  - `PATCH /api/activities/{activity_id}` — częściowa aktualizacja, ten
    sam zestaw pól, 404 dla nieznanego id; `Depends(require_admin)`.
  - `DELETE /api/activities/{activity_id}` — trwałe usunięcie; rozważ,
    co dzieje się z zależnymi wierszami `favorites`/`ratings` (zadanie
    08/09 dodają FK do `activities.id`) — usuń je kaskadowo w tej samej
    transakcji, zamiast pozostawiać osierocone wiersze lub zwracać błąd
    naruszenia ograniczenia FK. Udokumentuj tę kaskadę wprost w
    docstringu trasy, ponieważ to destrukcyjny, trudny do odwrócenia
    efekt uboczny.
- Odpowiedź `UserPublic`/`me()` w `backend/src/apps/users/routes.py`:
  dołącz `is_admin`, tak aby frontend wiedział, kiedy pokazać link
  nawigacyjny do panelu administracyjnego.
- Testy: `require_admin` zwraca 403 dla nie-administratora, 200 dla
  administratora; pełny cykl tworzenia/aktualizacji/usuwania dla każdego
  pola; usunięcie kaskadowo usuwa ulubione/oceny dla danej aktywności.

#### Zmiany w danych

- `make make-migrations` — `add_column("users", "is_admin", ...,
  server_default="false")`.
- `make migrate`.
- **Uwaga dotycząca wdrożenia:** żaden użytkownik nie jest
  administratorem od razu po tej migracji. Nadanie pierwszego
  administratora wymaga jednego ręcznego kroku — udokumentuj go w opisie
  PR tego zadania (celowo nie jako przepływ w UI: samoobsługowy przycisk
  „zostań administratorem” zaprzeczałby sensowi tego zabezpieczenia):
  jednorazowe uruchomienie `UPDATE users SET is_admin = true WHERE email
  = '<ty>';` na docelowej bazie danych po wdrożeniu.

#### Zmiany we frontendzie

- `frontend/src/auth/types.ts`: dodaj `is_admin: boolean` do `MeUser`.
- Nowa `pages/AdminActivitiesPage.tsx` pod trasą `/admin/activities`
  (routing z zadania 02), renderowana tylko, gdy `currentUser.is_admin`
  — pokaż przetłumaczony stan „brak autoryzacji” (nie surowe 403) dla
  każdego innego zalogowanego użytkownika, który wejdzie tam
  bezpośrednio, i przekieruj niezalogowanych użytkowników najpierw do
  logowania.
  - Tabela/lista wszystkich aktywności z akcjami edycji i usuwania.
  - Formularz (tworzenie + edycja, ten sam komponent) obejmujący każde
    pole: nazwa, opis, adres, kategoria (wybór z sugerowanego zestawu z
    zadania 01), przełącznik wnętrze/na zewnątrz, czas trwania (liczba,
    minuty), min/max wieku (dwa pola liczbowe), miasto (tekst — tutaj
    dowolne wprowadzanie, w przeciwieństwie do tylko-do-odczytu listy
    rozwijanej z zadania 05, ponieważ to administratorzy tworzą nowe
    miasta), szerokość/długość geograficzna (dwa pola liczbowe — zwykłe
    wprowadzanie liczb wystarcza dla MVP; bez wybierania na mapie),
    dopasowanie do pogody (wybór: dowolna/tylko słoneczna/przyjazna
    deszczowi).
  - Usuwanie wymaga wbudowanego kroku potwierdzenia (nie cichego
    usunięcia jednym kliknięciem), biorąc pod uwagę efekt uboczny
    kaskadowego usuwania z zadania 09/08.
- Link nawigacyjny „Admin” we wspólnym nagłówku `Layout`, widoczny tylko,
  gdy `currentUser?.is_admin`.
- i18n: `admin.title`, `.notAuthorized`, `.create`, `.edit`, `.delete`,
  `.deleteConfirm`, plus etykiety pól wykorzystujące istniejące etykiety
  filtrów/aktywności z zadań 01–07, tam gdzie już istnieją (nie duplikuj
  kluczy, które już istnieją dla „wnętrze”, „kategoria” itd.) —
  wszystkie cztery pliki lokalizacji.

#### Poza zakresem

- Import masowy (CSV/JSON) — wprowadzanie pojedyncze jest akceptowalne,
  by odblokować zimny start; wróć do tematu, jeśli ręczne wprowadzanie
  okaże się zbyt wolne, gdy poznamy realny wolumen treści.
- Role/uprawnienia poza pojedynczą flagą `is_admin` (np. moderatorzy
  regionalni) — niepotrzebne przy obecnej skali.
- Wybieranie współrzędnych na mapie — zwykłe pola liczbowe lat/lng
  wystarczają dla MVP; administrator może sprawdzić współrzędne
  zewnętrznie.
- Pole `suitable_for_gender` z zadania 14 — to zadanie zostanie
  dostarczone po tym i powinno rozszerzyć ten formularz o jedno
  dodatkowe pole, gdy się pojawi, zamiast uzależniać to zadanie od
  jeszcze nieistniejącej kolumny.

#### Kryteria akceptacji

- Niezalogowany-jako-administrator zalogowany użytkownik wchodzący na
  `/admin/activities` widzi przetłumaczony komunikat „brak autoryzacji”,
  a nie surowy JSON czy błąd aplikacji; odpowiednie wywołania API
  zwracają 403, a nie 404 czy 500.
- Administrator może utworzyć aktywność z wypełnionymi wszystkimi
  polami i natychmiast zobaczyć ją na publicznej stronie przeglądania,
  z poprawnym działaniem wszystkich filtrów (zadania 03–07).
- Usunięcie aktywności, która ma ulubione i oceny, kończy się sukcesem i
  nie pozostawia osieroconych wierszy w żadnej z tabel.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
