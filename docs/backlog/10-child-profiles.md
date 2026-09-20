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

---

## Tłumaczenie (PL)

### 10. Profile dzieci

#### Podsumowanie

Pozwól zalogowanym rodzicom zapisać na koncie jeden lub więcej profili
dzieci (imię + data urodzenia), ze stroną zarządzania „Moje dzieci”. To
zadanie nie zmienia jeszcze doświadczenia przeglądania — to podłączenie
jest zadaniem 11 — to wyłącznie ekran wprowadzania/zarządzania danymi.

#### Dlaczego

Ręczne wpisywanie wieku w filtrze (zadanie 04) działa, ale prawdziwym
wyróżnikiem produktu według podsumowania odkrycia („silnik decyzyjny, a
nie kolejny katalog”) jest personalizacja — wiedza, że rodzic ma 4-latka,
bez konieczności wpisywania tego przy każdej wizycie. To zadanie
zapisuje te dane; zadanie 11 je wykorzystuje.

#### Zmiany w backendzie

- Nowa aplikacja `backend/src/apps/children/`:
  - `models.py` — tabela `ChildProfile` o nazwie `child_profiles`:
    `id: UUID` pk, `user_id: UUID` (`foreign_key="users.id"`,
    indeksowane), `name: str` (`max_length=100`), `birth_date: date`,
    `created_at: datetime`.
  - `api_errors.py` — `ApiErrorCode.child_not_found`.
  - `routes.py` — `APIRouter(prefix="/api/children", tags=["children"])`,
    wszystkie trasy wymagają `Depends(get_current_user)` (z
    `users/deps.py` z zadania 08):
    - `GET /api/children` — lista dzieci bieżącego użytkownika, każde z
      wyliczonym `age_years` obliczonym po stronie serwera z
      `birth_date` (tak aby frontend nigdy nie musiał ponownie
      implementować liczenia wieku — wykorzystywane ponownie w zadaniu
      11).
    - `POST /api/children` — treść `{"name": str, "birth_date": date}`.
      Odrzuć przyszłą `birth_date` (422) oraz nierealistyczny wiek (np.
      `birth_date` starsza niż 21 lat — zakres tego produktu to
      aktywności dla dzieci; miękki sufit walidacji zapobiega śmieciowym
      danym, a nie jest twardą regułą produktową).
    - `PATCH /api/children/{child_id}` — częściowa aktualizacja,
      ograniczona do wywołującego; 404 (`child_not_found`), jeśli id nie
      należy do niego (nie ujawniaj istnienia cudzych wierszy przez
      rozróżnienie 403 vs 404).
    - `DELETE /api/children/{child_id}` — ograniczone do wywołującego,
      idempotentne.
  - `__init__.py`
- Zarejestruj `children_router` w `backend/src/main.py`.
- Testy w `backend/src/apps/children/tests/test_routes.py`: operacje CRUD
  ograniczone do wywołującego (jeden użytkownik nie może
  czytać/edytować/usuwać cudzego dziecka), poprawność wyliczania wieku,
  błędy walidacji dla przyszłych/nierealistycznych dat urodzenia.

#### Zmiany w danych

- `make make-migrations` — nowa tabela `child_profiles`.
- `make migrate`.

#### Zmiany we frontendzie

- Dodaj `@mantine/dates` i `dayjs` do `frontend/package.json` (pole daty
  Mantine potrzebuje obu; jeszcze niezainstalowane).
- Nowa `pages/ChildrenPage.tsx` pod trasą `/children` (routing z zadania
  02):
  - Lista dzieci zalogowanego użytkownika jako karty (imię, wyliczony
    wiek).
  - Formularz „Dodaj dziecko”: pole tekstowe imienia + `DateInput` na
    datę urodzenia.
  - Akcje edycji/usuwania dla każdego dziecka.
  - Trasa jest dostępna/pokazywana w nawigacji tylko po zalogowaniu;
    przekieruj (lub pokaż monit o zalogowanie), jeśli niezalogowany
    użytkownik wejdzie bezpośrednio na `/children`.
- Link nawigacyjny „Moje dzieci” we wspólnym nagłówku `Layout`, widoczny
  po zalogowaniu (obok linku „Ulubione” dodanego w zadaniu 08).
- i18n: `children.title`, `.addChild`, `.name`, `.birthDate`, `.age`,
  `.empty`, `.deleteConfirm` — wszystkie cztery pliki lokalizacji.

#### Poza zakresem

- Podłączenie profili dzieci do filtrowania/rekomendacji aktywności —
  to zadanie 11.
- Powiązanie profilu dziecka z wysłaną recenzją w celu zanonimizowanego
  kontekstu — to zadanie 16.
- Zdjęcia lub dodatkowe atrybuty dziecka (zainteresowania itp.) poza
  imieniem i datą urodzenia — utrzymane minimalnie dla MVP.

#### Kryteria akceptacji

- Użytkownik może dodać, edytować i usuwać dzieci; dzieci innego
  zalogowanego użytkownika nigdy nie pojawiają się na jego liście ani
  nie są przez niego edytowalne.
- `age_years` zwracane przez API zgadza się z datą urodzenia (np. data
  urodzenia dokładnie 4 lata temu od dziś raportuje `4`).
- Niezalogowany dostęp do `/children` nie powoduje błędu aplikacji — tylko
  monit o zalogowanie.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
