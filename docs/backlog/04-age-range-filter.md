---
status: todo
---

# 04. Age range filter

## Summary

Add an age range to each activity and let parents filter the browse list by
a single age ("suitable for my 4-year-old"). This is the "age-based
recommendations" line item from the discovery summary's MVP scope, in its
simplest form — a manual age input, not yet tied to a saved child profile
(that personalization layer is task 11, built on top of this).

## Why

Age-appropriateness is the single most requested filter for a parenting
product and doesn't need any other feature to be useful on its own.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `age_min_years: int | None` — nullable (no known lower bound).
  - `age_max_years: int | None` — nullable (no known upper bound).
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional query param `age_years: int | None`. When set, filter to
  rows where `(age_min_years IS NULL OR age_min_years <= age_years)` AND
  `(age_max_years IS NULL OR age_max_years >= age_years)` — i.e. an unset
  bound means "no restriction on that side," not "excludes everything."
  Combine with `AND` alongside the task 03 filters.
- Add validation: reject (`422`, standard FastAPI validation) `age_years`
  outside `0–17`.
- Extend `test_routes.py`: activity with no age bounds matches any
  `age_years`; activity with both bounds only matches inside the range;
  activity with only one bound set is correctly open-ended on the other
  side.

## Data changes

- `make make-migrations` — both new columns nullable, no backfill needed.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx` filter bar: add a `NumberInput` "Child's age" (0–17,
  empty = no filter), wired into the same `useSearchParams`-backed filter
  state as task 03 (`?age=4`).
- Show an age range badge on each card/detail page, e.g. "Ages 3–8" or
  "Ages 3+" or "All ages" when both bounds are null.
- i18n: `filters.age.label`, `activities.ageRange.bounded` (`"Ages {{min}}–{{max}}"`),
  `.minOnly` (`"Ages {{min}}+"`), `.maxOnly` (`"Up to age {{max}}"`),
  `.any` (`"All ages"`) — added to all four locale files.

## Out of scope

- Saved child profiles (task 10) and auto-applying a child's age (task 11).
- Multiple age ranges per activity (e.g. different sessions for different
  ages) — one range per activity is enough for MVP.
- Gender-suitability filtering — a separate dimension, covered by task 14.

## Acceptance criteria

- `GET /api/activities?age_years=4` includes an activity with
  `age_min_years=3, age_max_years=8`, excludes one with
  `age_min_years=10`, and includes one with both bounds null.
- The age input combines correctly with the indoor/outdoor/duration filters
  from task 03 (all present in the query string at once narrows correctly).
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 04. Filtr zakresu wieku

#### Podsumowanie

Dodaj zakres wieku do każdej aktywności i pozwól rodzicom filtrować listę
przeglądania po pojedynczym wieku („odpowiednie dla mojego 4-latka”). To
najprostsza forma pozycji „rekomendacje oparte na wieku” z zakresu MVP w
podsumowaniu odkrycia — ręczne pole wieku, jeszcze niepowiązane z
zapisanym profilem dziecka (ta warstwa personalizacji to zadanie 11,
budowane na tym zadaniu).

#### Dlaczego

Odpowiedniość wiekowa to najczęściej oczekiwany filtr w produkcie dla
rodziców i jest użyteczna sama w sobie, bez potrzeby żadnej innej funkcji.

#### Zmiany w backendzie

- `backend/src/apps/activities/models.py` — dodaj do `Activity`:
  - `age_min_years: int | None` — puste (brak znanej dolnej granicy).
  - `age_max_years: int | None` — puste (brak znanej górnej granicy).
- `backend/src/apps/activities/routes.py` — rozszerz `GET /api/activities`
  o opcjonalny parametr zapytania `age_years: int | None`. Gdy ustawiony,
  filtruj wiersze, gdzie `(age_min_years IS NULL OR age_min_years <=
  age_years)` ORAZ `(age_max_years IS NULL OR age_max_years >=
  age_years)` — czyli brak granicy oznacza „brak ograniczenia po tej
  stronie”, a nie „wyklucza wszystko”. Połącz operatorem `AND` z filtrami
  z zadania 03.
- Dodaj walidację: odrzucaj (`422`, standardowa walidacja FastAPI)
  `age_years` spoza zakresu `0–17`.
- Rozszerz `test_routes.py`: aktywność bez granic wieku pasuje do każdego
  `age_years`; aktywność z obiema granicami pasuje tylko wewnątrz
  zakresu; aktywność z ustawioną tylko jedną granicą jest poprawnie
  otwarta po drugiej stronie.

#### Zmiany w danych

- `make make-migrations` — obie nowe kolumny puste, bez potrzeby
  uzupełniania danych wstecz.
- `make migrate`.

#### Zmiany we frontendzie

- Pasek filtrów w `BrowsePage.tsx`: dodaj `NumberInput` „Wiek dziecka”
  (0–17, puste = brak filtra), podpięty do tego samego stanu filtrów
  opartego na `useSearchParams` co zadanie 03 (`?age=4`).
- Pokaż znacznik zakresu wieku na każdej karcie/stronie szczegółów, np.
  „Wiek 3–8” albo „Wiek 3+” albo „Wszystkie grupy wiekowe”, gdy obie
  granice są puste.
- i18n: `filters.age.label`, `activities.ageRange.bounded`
  (`"Ages {{min}}–{{max}}"`), `.minOnly` (`"Ages {{min}}+"`), `.maxOnly`
  (`"Up to age {{max}}"`), `.any` (`"All ages"`) — dodane do wszystkich
  czterech plików lokalizacji.

#### Poza zakresem

- Zapisane profile dzieci (zadanie 10) i automatyczne stosowanie wieku
  dziecka (zadanie 11).
- Wiele zakresów wieku na jedną aktywność (np. różne sesje dla różnych
  grup wiekowych) — jeden zakres na aktywność wystarcza dla MVP.
- Filtrowanie po płci — osobny wymiar, objęty zadaniem 14.

#### Kryteria akceptacji

- `GET /api/activities?age_years=4` obejmuje aktywność z
  `age_min_years=3, age_max_years=8`, wyklucza tę z `age_min_years=10`
  oraz obejmuje tę z obiema granicami pustymi.
- Pole wieku poprawnie łączy się z filtrami wnętrze/na
  zewnątrz/czas trwania z zadania 03 (wszystkie obecne jednocześnie w
  query stringu poprawnie zawężają wyniki).
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
