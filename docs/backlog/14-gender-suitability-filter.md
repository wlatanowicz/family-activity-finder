---
status: todo
---

# 14. Gender-suitability filter

## Summary

Add an optional gender-suitability tag to each activity (e.g. a dance class
marked "girls," a football clinic marked "boys," most activities left
unmarked/"all") and let parents filter the browse list by it. This is the
second half of the "filtrowanie po wieku/płci" (age/gender filtering) idea
from `docs/backlog/IDEAS.md` — the age half already shipped as task 04.

## Why

Task 04 covers age but not gender; `IDEAS.md` calls out both in the same
idea. Some activities in practice are marketed/run as gender-specific
(certain dance, sports, or scouting programs), so surfacing it as a filter
avoids parents finding activities that turn out not to fit their child once
they show up.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `suitable_for_gender: str | None` — `max_length=10`, nullable. Nullable
    means "all genders" (the common case) — do not require every activity
    to set it. Validate against a `Literal`/constant list at the API layer
    (mirroring task 01's `category` approach): `girls`, `boys`, `all`. A
    row can also simply be `None`, treated identically to `all`.
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional query param `gender: Literal["girls", "boys"] | None`. When
  set, return rows where `suitable_for_gender IS NULL OR
  suitable_for_gender IN ('all', <gender>)` — i.e. gender-specific
  activities for the other gender are excluded, but unmarked/"all"
  activities always match. Combine with `AND` alongside every other filter
  (tasks 03–07).
- Extend `test_routes.py`: `gender=girls` includes activities marked
  `girls` and `all` and null, excludes `boys`; no `gender` param returns
  everything unfiltered.

## Data changes

- `make make-migrations` — new nullable column, no backfill needed
  (existing/new rows default to "all" via `None`).
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx` filter bar: add a `SegmentedControl`/`Select` "Suitable
  for" with options "All," "Girls," "Boys" (default "All," which means no
  filter applied — not "show only unmarked activities"), wired into the
  same `useSearchParams`-backed filter state as tasks 03–04 (`?gender=girls`).
- Show a badge on activity cards/detail page only when
  `suitable_for_gender` is set to `girls` or `boys` (omit the badge
  entirely for `all`/null, matching the existing convention from task 09's
  rating badge of not cluttering the common case).
- i18n: `filters.gender.label`, `.all`, `.girls`, `.boys`,
  `activities.genderBadge.girls`, `.boys` — all four locale files.

## Out of scope

- Any inference/auto-tagging of gender suitability from activity name or
  category — it's a plain admin-entered field (task 12 extends the admin
  form to cover it), not a heuristic.
- Non-binary/other categories beyond "girls / boys / all" — kept to the
  three values named in the source idea; revisit if real content needs
  finer granularity.

## Acceptance criteria

- `GET /api/activities?gender=girls` includes activities marked `girls` or
  `all` or with the field unset, and excludes ones marked `boys`.
- The gender filter combines correctly with the age (task 04) and
  indoor/outdoor/duration (task 03) filters in the same query string.
- Cards for activities with `suitable_for_gender` unset/`all` show no
  gender badge.
- `make check` passes; `cd frontend && npm run build` succeeds.

## Follow-up in other tasks

- Task 12 (admin activity management) should add `suitable_for_gender` to
  its create/edit form field set alongside the other tasks 01–07 columns.

---

## Tłumaczenie (PL)

### 14. Filtr dopasowania do płci

#### Podsumowanie

Dodaj opcjonalny znacznik dopasowania do płci dla każdej aktywności (np.
zajęcia taneczne oznaczone „dziewczynki”, klinika piłkarska oznaczona
„chłopcy”, większość aktywności pozostaje nieoznaczona/„wszyscy”) i
pozwól rodzicom filtrować po nim listę przeglądania. To druga połowa
pomysłu „filtrowanie po wieku/płci” z `docs/backlog/IDEAS.md` — połowa
dotycząca wieku została już dostarczona jako zadanie 04.

#### Dlaczego

Zadanie 04 obejmuje wiek, ale nie płeć; `IDEAS.md` wymienia oba w tym
samym pomyśle. Niektóre aktywności w praktyce są marketowane/prowadzone
jako specyficzne dla płci (niektóre zajęcia taneczne, sportowe czy
harcerskie), więc udostępnienie tego jako filtra pozwala uniknąć sytuacji,
w której rodzic znajduje aktywność, która na miejscu okazuje się
niedopasowana do jego dziecka.

#### Zmiany w backendzie

- `backend/src/apps/activities/models.py` — dodaj do `Activity`:
  - `suitable_for_gender: str | None` — `max_length=10`, puste. Puste
    oznacza „wszystkie płcie” (najczęstszy przypadek) — nie wymagaj, aby
    każda aktywność miała to ustawione. Waliduj względem listy
    `Literal`/stałych na poziomie API (odzwierciedlając podejście
    `category` z zadania 01): `girls`, `boys`, `all`. Wiersz może też
    mieć po prostu `None`, traktowane identycznie jak `all`.
- `backend/src/apps/activities/routes.py` — rozszerz `GET /api/activities`
  o opcjonalny parametr zapytania `gender: Literal["girls", "boys"] |
  None`. Gdy ustawiony, zwróć wiersze, gdzie `suitable_for_gender IS
  NULL OR suitable_for_gender IN ('all', <gender>)` — czyli aktywności
  specyficzne dla przeciwnej płci są wykluczane, ale nieoznaczone/„all”
  zawsze pasują. Połącz operatorem `AND` z każdym innym filtrem (zadania
  03–07).
- Rozszerz `test_routes.py`: `gender=girls` obejmuje aktywności oznaczone
  `girls`, `all` i puste, wyklucza `boys`; brak parametru `gender` zwraca
  wszystko bez filtrowania.

#### Zmiany w danych

- `make make-migrations` — nowa pusta kolumna, bez potrzeby uzupełniania
  danych wstecz (istniejące/nowe wiersze domyślnie są „all” poprzez
  `None`).
- `make migrate`.

#### Zmiany we frontendzie

- Pasek filtrów w `BrowsePage.tsx`: dodaj `SegmentedControl`/`Select`
  „Dopasowane do” z opcjami „Wszyscy”, „Dziewczynki”, „Chłopcy”
  (domyślnie „Wszyscy”, co oznacza brak zastosowanego filtra — nie
  „pokaż tylko nieoznaczone aktywności”), podpięty do tego samego stanu
  filtrów `useSearchParams` co zadania 03–04 (`?gender=girls`).
- Pokaż znacznik na kartach aktywności/stronie szczegółów tylko, gdy
  `suitable_for_gender` jest ustawione na `girls` lub `boys` (całkowicie
  pomiń znacznik dla `all`/pustego, zgodnie z istniejącą konwencją
  znacznika oceny z zadania 09, by nie zaśmiecać typowego przypadku).
- i18n: `filters.gender.label`, `.all`, `.girls`, `.boys`,
  `activities.genderBadge.girls`, `.boys` — wszystkie cztery pliki
  lokalizacji.

#### Poza zakresem

- Jakiekolwiek wnioskowanie/automatyczne oznaczanie dopasowania do płci
  na podstawie nazwy lub kategorii aktywności — to zwykłe pole
  wprowadzane przez administratora (zadanie 12 rozszerza formularz
  administracyjny o nie obsługę), a nie heurystyka.
- Kategorie niebinarne/inne poza „dziewczynki / chłopcy / wszyscy” —
  ograniczone do trzech wartości wymienionych w źródłowym pomyśle; wróć
  do tematu, jeśli realna treść będzie wymagać większej granulacji.

#### Kryteria akceptacji

- `GET /api/activities?gender=girls` obejmuje aktywności oznaczone
  `girls` lub `all` lub z nieustawionym polem, i wyklucza oznaczone
  `boys`.
- Filtr płci poprawnie łączy się z filtrami wieku (zadanie 04) i
  wnętrze/na zewnątrz/czas trwania (zadanie 03) w tym samym query
  stringu.
- Karty aktywności z nieustawionym/`all` `suitable_for_gender` nie
  pokazują znacznika płci.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.

#### Kontynuacja w innych zadaniach

- Zadanie 12 (zarządzanie aktywnościami przez administratora) powinno
  dodać `suitable_for_gender` do zestawu pól formularza
  tworzenia/edycji, obok pozostałych kolumn z zadań 01–07.
