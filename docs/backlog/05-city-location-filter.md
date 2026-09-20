---
status: todo
---

# 05. City location filter

## Summary

Add a city/region field to activities and let parents filter the browse
list down to a specific city, via a dropdown populated from the activities
actually in the database. This is the first, simplest slice of the
discovery summary's "location-aware search" — a text match on city, no
maps or distance math yet (that's task 06).

## Why

Most parents search "things to do in [my city]," not by geographic radius.
A city filter is useful standalone and is a prerequisite-free step before
the geolocation/distance feature in task 06.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `city: str | None` — `max_length=100`, indexed. Nullable because not
    every activity will have this back-filled immediately after migration.
- `backend/src/apps/activities/routes.py`:
  - Extend `GET /api/activities` with optional query param `city: str | None`
    — exact, case-insensitive match (normalize both sides with
    `func.lower()` or an equivalent) against `Activity.city`. Combine with
    `AND` alongside existing filters.
  - Add `GET /api/activities/cities` — returns
    `{"cities": ["Berlin", "Munich", ...]}`, the distinct, non-null `city`
    values currently in the table, sorted alphabetically. Powers the
    frontend dropdown without hardcoding a city list.
- Extend `test_routes.py`: `city` filter matches case-insensitively;
  `/cities` returns distinct sorted values and omits nulls.

## Data changes

- `make make-migrations` — nullable `city` column, indexed.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx`: fetch `GET /api/activities/cities` on mount, populate a
  `Select` ("All cities" + each returned city), wired into the same
  `useSearchParams` filter state as tasks 03–04 (`?city=Berlin`).
- Show the city on each activity card (alongside the existing address
  line) and on the detail page.
- i18n: `filters.city.label`, `filters.city.all` — added to all four locale
  files.

## Out of scope

- Free-text/fuzzy city search — an exact-match dropdown is enough since the
  list is generated from real data, not user-typed.
- Distance-based "near me" sorting (task 06).

## Acceptance criteria

- `GET /api/activities/cities` reflects only cities that currently have at
  least one activity.
- `GET /api/activities?city=berlin` matches activities with `city="Berlin"`.
- The city filter composes correctly with age/indoor/duration filters from
  prior tasks.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 05. Filtr lokalizacji (miasto)

#### Podsumowanie

Dodaj pole miasta/regionu do aktywności i pozwól rodzicom filtrować listę
przeglądania do konkretnego miasta, przez listę rozwijaną wypełnianą
danymi z aktywności rzeczywiście obecnych w bazie. To pierwszy, najprostszy
wycinek „wyszukiwania uwzględniającego lokalizację” z podsumowania
odkrycia — dopasowanie tekstowe po mieście, jeszcze bez map i liczenia
odległości (to zadanie 06).

#### Dlaczego

Większość rodziców szuka „co robić w [moim mieście]”, a nie w promieniu
geograficznym. Filtr miasta jest użyteczny samodzielnie i stanowi krok bez
żadnych wymagań wstępnych przed funkcją geolokalizacji/odległości z
zadania 06.

#### Zmiany w backendzie

- `backend/src/apps/activities/models.py` — dodaj do `Activity`:
  - `city: str | None` — `max_length=100`, indeksowane. Puste, ponieważ
    nie każda aktywność będzie miała to pole uzupełnione od razu po
    migracji.
- `backend/src/apps/activities/routes.py`:
  - Rozszerz `GET /api/activities` o opcjonalny parametr zapytania
    `city: str | None` — dokładne dopasowanie, bez rozróżniania
    wielkości liter (znormalizuj obie strony za pomocą `func.lower()`
    lub odpowiednika) względem `Activity.city`. Połącz operatorem `AND`
    z istniejącymi filtrami.
  - Dodaj `GET /api/activities/cities` — zwraca
    `{"cities": ["Berlin", "Munich", ...]}`, czyli unikalne, niepuste
    wartości `city` obecne aktualnie w tabeli, posortowane alfabetycznie.
    Zasila listę rozwijaną we frontendzie bez zaszywania listy miast na
    stałe w kodzie.
- Rozszerz `test_routes.py`: filtr `city` dopasowuje bez względu na
  wielkość liter; `/cities` zwraca unikalne, posortowane wartości i
  pomija puste.

#### Zmiany w danych

- `make make-migrations` — puste, indeksowane pole `city`.
- `make migrate`.

#### Zmiany we frontendzie

- `BrowsePage.tsx`: pobierz `GET /api/activities/cities` przy
  zamontowaniu, wypełnij `Select` („Wszystkie miasta” + każde zwrócone
  miasto), podpięty do tego samego stanu filtrów `useSearchParams` co
  zadania 03–04 (`?city=Berlin`).
- Pokaż miasto na każdej karcie aktywności (obok istniejącej linii
  adresu) oraz na stronie szczegółów.
- i18n: `filters.city.label`, `filters.city.all` — dodane do wszystkich
  czterech plików lokalizacji.

#### Poza zakresem

- Dowolne/rozmyte wyszukiwanie tekstowe miasta — lista rozwijana z
  dokładnym dopasowaniem wystarcza, ponieważ jest generowana z
  rzeczywistych danych, a nie wpisywana przez użytkownika.
- Sortowanie „w pobliżu” oparte na odległości (zadanie 06).

#### Kryteria akceptacji

- `GET /api/activities/cities` odzwierciedla wyłącznie miasta, które
  aktualnie mają co najmniej jedną aktywność.
- `GET /api/activities?city=berlin` dopasowuje aktywności z
  `city="Berlin"`.
- Filtr miasta poprawnie współdziała z filtrami wieku/wnętrze/czasu
  trwania z poprzednich zadań.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
