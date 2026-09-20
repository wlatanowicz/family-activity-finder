---
status: todo
---

# 03. Indoor/outdoor and duration filters

## Summary

Add the "indoor/outdoor" and "duration" filters called out in the discovery
summary's MVP scope. Parents can narrow the browse list to activities that
fit whether they're stuck inside or have a specific amount of free time.

## Why

These two are grouped in the discovery summary as a single MVP filter set,
and both are simple scalar columns with no external dependency (unlike
weather or location) — the cheapest filters to ship first.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `is_indoor: bool` — not nullable, default `True` (safe default so
    existing rows don't need backfill logic beyond the default).
  - `duration_minutes: int | None` — approximate typical visit length;
    nullable because not every activity has a fixed duration (e.g. an
    open-ended park).
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional query params:
  - `indoor: bool | None` — when set, filter `is_indoor == indoor`.
  - `max_duration_minutes: int | None` — when set, filter
    `duration_minutes <= max_duration_minutes` (rows with `NULL` duration
    are excluded when this filter is active, since "fits in 30 minutes" is
    unknown for them).
  - Both params combine with `AND` and with each other; no params = no
    filtering (current behavior preserved).
- Update `backend/src/apps/activities/tests/test_routes.py` with cases for
  each param alone and combined.

## Data changes

- `make make-migrations` then review: `add_column("activities", "is_indoor", ...)`
  with `server_default="true"` so the migration backfills existing rows
  without a null gap, and `add_column("activities", "duration_minutes", ...)`
  nullable.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx`: add a filter bar above the list:
  - Indoor/Outdoor/Any — `SegmentedControl` (Mantine), three options.
  - Duration — `Select` with buckets: "Any", "Under 30 min", "30–60 min",
    "1–2 hours", "2+ hours" (map bucket to `max_duration_minutes`: 30, 60,
    120, and "Any"/"2+ hours" sends no param or a high ceiling —
    "2+ hours" should NOT send `max_duration_minutes`, since it means "no
    upper bound", not "under some max"; only the first three buckets map to
    the param).
  - Filter state lives in the URL query string (e.g. `?indoor=true&maxDuration=60`)
    via `useSearchParams` from `react-router-dom`, so filtered views are
    shareable/bookmarkable and survive a refresh.
  - Re-fetch `GET /api/activities` whenever the query string changes.
- Show an `is_indoor` badge ("Indoor"/"Outdoor") and duration (when set) on
  each activity card and on the detail page from task 02.
- i18n: add `filters.indoor`, `filters.outdoor`, `filters.any`,
  `filters.duration.*` bucket labels to all four locale files.

## Out of scope

- Age, location, weather filters (tasks 04–07) — this task only wires up
  indoor/outdoor and duration.
- Persisting a user's preferred filters across sessions.

## Acceptance criteria

- `GET /api/activities?indoor=true` returns only indoor activities;
  `?indoor=false` only outdoor; omitted returns all.
- `GET /api/activities?max_duration_minutes=30` excludes activities with
  `duration_minutes` null or `> 30`.
- Changing the filter bar updates the URL and the list without a full page
  reload; reloading the page with a filtered URL reproduces the same
  filtered list.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 03. Filtry wnętrze/na zewnątrz i czas trwania

#### Podsumowanie

Dodaj filtry „wnętrze/na zewnątrz” oraz „czas trwania” wskazane w zakresie
MVP z podsumowania odkrycia. Rodzice mogą zawęzić listę przeglądania do
aktywności pasujących do tego, czy utknęli w domu, czy mają konkretną
ilość wolnego czasu.

#### Dlaczego

Te dwa filtry są zgrupowane w podsumowaniu odkrycia jako jeden zestaw
filtrów MVP, a oba to proste kolumny skalarne bez zależności zewnętrznych
(w przeciwieństwie do pogody czy lokalizacji) — najtańsze filtry do
wdrożenia jako pierwsze.

#### Zmiany w backendzie

- `backend/src/apps/activities/models.py` — dodaj do `Activity`:
  - `is_indoor: bool` — niepuste, domyślnie `True` (bezpieczna wartość
    domyślna, dzięki czemu istniejące wiersze nie wymagają dodatkowej
    logiki uzupełniania poza samą wartością domyślną).
  - `duration_minutes: int | None` — przybliżony typowy czas trwania
    wizyty; puste, ponieważ nie każda aktywność ma stały czas trwania
    (np. park bez określonego czasu).
- `backend/src/apps/activities/routes.py` — rozszerz `GET /api/activities`
  o opcjonalne parametry zapytania:
  - `indoor: bool | None` — gdy ustawione, filtruje `is_indoor == indoor`.
  - `max_duration_minutes: int | None` — gdy ustawione, filtruje
    `duration_minutes <= max_duration_minutes` (wiersze z `NULL` w czasie
    trwania są wykluczane, gdy ten filtr jest aktywny, ponieważ nie
    wiadomo, czy „mieszczą się w 30 minutach”).
  - Oba parametry łączą się operatorem `AND`, zarówno ze sobą, jak i z
    innymi filtrami; brak parametrów = brak filtrowania (zachowane
    bieżące zachowanie).
- Zaktualizuj `backend/src/apps/activities/tests/test_routes.py` o
  przypadki dla każdego parametru osobno i łącznie.

#### Zmiany w danych

- `make make-migrations`, a następnie sprawdź:
  `add_column("activities", "is_indoor", ...)` z `server_default="true"`,
  tak aby migracja uzupełniła istniejące wiersze bez pustych wartości,
  oraz `add_column("activities", "duration_minutes", ...)` jako pole
  puste.
- `make migrate`.

#### Zmiany we frontendzie

- `BrowsePage.tsx`: dodaj pasek filtrów nad listą:
  - Wnętrze/Na zewnątrz/Dowolne — `SegmentedControl` (Mantine), trzy
    opcje.
  - Czas trwania — `Select` z przedziałami: „Dowolny”, „Poniżej 30 min”,
    „30–60 min”, „1–2 godziny”, „2+ godziny” (mapowanie przedziału na
    `max_duration_minutes`: 30, 60, 120, a „Dowolny”/„2+ godziny” nie
    wysyłają parametru lub wysyłają wysoki limit — „2+ godziny” NIE
    powinno wysyłać `max_duration_minutes`, bo oznacza „brak górnego
    limitu”, a nie „poniżej jakiegoś maksimum”; tylko pierwsze trzy
    przedziały mapują się na ten parametr).
  - Stan filtrów żyje w query stringu URL (np.
    `?indoor=true&maxDuration=60`) za pomocą `useSearchParams` z
    `react-router-dom`, dzięki czemu przefiltrowane widoki można
    udostępniać/zapisywać w zakładkach i przetrwają odświeżenie.
  - Ponownie pobieraj `GET /api/activities` za każdym razem, gdy zmienia
    się query string.
- Pokaż znacznik `is_indoor` („Wnętrze”/„Na zewnątrz”) oraz czas trwania
  (gdy ustawiony) na każdej karcie aktywności i na stronie szczegółów z
  zadania 02.
- i18n: dodaj etykiety `filters.indoor`, `filters.outdoor`, `filters.any`,
  `filters.duration.*` dla przedziałów do wszystkich czterech plików
  lokalizacji.

#### Poza zakresem

- Filtry wieku, lokalizacji i pogody (zadania 04–07) — to zadanie podpina
  tylko wnętrze/na zewnątrz i czas trwania.
- Zapamiętywanie preferowanych filtrów użytkownika między sesjami.

#### Kryteria akceptacji

- `GET /api/activities?indoor=true` zwraca tylko aktywności w
  pomieszczeniach; `?indoor=false` — tylko na zewnątrz; brak parametru
  zwraca wszystkie.
- `GET /api/activities?max_duration_minutes=30` wyklucza aktywności z
  `duration_minutes` pustym lub `> 30`.
- Zmiana paska filtrów aktualizuje URL i listę bez pełnego przeładowania
  strony; odświeżenie strony z przefiltrowanym URL-em odtwarza tę samą
  przefiltrowaną listę.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
