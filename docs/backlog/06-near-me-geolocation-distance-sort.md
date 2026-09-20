---
status: todo
---

# 06. "Near me" geolocation distance sort

## Summary

Add coordinates to activities and let parents sort the browse list by
distance from their current location, using the browser's Geolocation API.
This is the second, richer slice of the discovery summary's "location-aware
search," building on the city filter from task 05.

## Why

"What's closest to me right now" is a distinct use case from "what's in
this city" (task 05) — e.g. a parent near a city border, or comparing two
close-by options within the same city. Coordinates are also needed
groundwork for any future map view.

## Backend changes

- `backend/src/apps/activities/models.py` — add to `Activity`:
  - `latitude: float | None`
  - `longitude: float | None`
  - Both nullable — coordinates get backfilled as activities are
    created/edited (task 12); rows without them simply can't be
    distance-sorted.
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional `lat: float | None`, `lng: float | None` (must be provided
  together — 422 if only one is set):
  - When both are provided, compute the haversine distance in Python
    between `(lat, lng)` and each row's `(latitude, longitude)`, attach it
    as `distance_km` in the response, and sort ascending by distance.
    Rows with null coordinates get `distance_km: null` and sort last (not
    excluded — a parent should still see them, just below located results).
  - Combines with all existing filters (`indoor`, `max_duration_minutes`,
    `age_years`, `city`) via `AND`; sorting by distance only kicks in when
    `lat`/`lng` are present, otherwise sort order is unchanged (current
    default, e.g. by name or `created_at`).
  - Put the haversine helper in `backend/src/apps/activities/geo.py` so
    it's unit-testable in isolation.
- Add `backend/src/apps/activities/tests/test_geo.py` (known-distance pairs,
  e.g. two coordinates ~1km apart) and extend `test_routes.py` for the
  sorting/null-handling behavior and the "only one of lat/lng" 422 case.

## Data changes

- `make make-migrations` — two new nullable float columns.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx`: add a "Use my location" button. On click, call
  `navigator.geolocation.getCurrentPosition`; on success, store
  `{lat, lng}` in component state and add them to the `useSearchParams`
  filter state (`?lat=...&lng=...`) so results resort; on denial/error,
  show a translated inline message and leave sorting unchanged (don't
  block the rest of the page).
  - This is a real device permission prompt — keep it strictly
    opt-in/behind the button; never request location automatically on
    page load.
- Show `distance_km` (formatted, e.g. "2.3 km away") on each card when
  present, once location is active.
- i18n: `filters.nearMe.button`, `filters.nearMe.denied`,
  `filters.nearMe.unsupported`, `activities.distanceAway` — all four locale
  files.

## Out of scope

- A map view — this task only adds list sorting, no visual map (see task
  13).
- Editing coordinates from the UI (task 12 handles data entry).
- Configurable radius cutoff (e.g. "within 10 km") — sort-only for MVP.

## Acceptance criteria

- With `lat`/`lng` supplied, activities with coordinates are returned
  nearest-first with a correct `distance_km`; activities without
  coordinates appear after all located ones.
- Supplying only `lat` (no `lng`) returns 422.
- Denying the browser location permission leaves the browse page usable
  (no crash, no blocked UI), with a translated explanation shown.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 06. Sortowanie po odległości „w pobliżu” (geolokalizacja)

#### Podsumowanie

Dodaj współrzędne do aktywności i pozwól rodzicom sortować listę
przeglądania po odległości od bieżącej lokalizacji, korzystając z API
geolokalizacji przeglądarki. To drugi, bogatszy wycinek „wyszukiwania
uwzględniającego lokalizację” z podsumowania odkrycia, budowany na filtrze
miasta z zadania 05.

#### Dlaczego

„Co jest najbliżej mnie teraz” to inny przypadek użycia niż „co jest w tym
mieście” (zadanie 05) — np. rodzic w pobliżu granicy miasta albo
porównujący dwie bliskie opcje w tym samym mieście. Współrzędne są też
niezbędnym fundamentem pod ewentualny przyszły widok mapy.

#### Zmiany w backendzie

- `backend/src/apps/activities/models.py` — dodaj do `Activity`:
  - `latitude: float | None`
  - `longitude: float | None`
  - Obie puste — współrzędne są uzupełniane wraz z tworzeniem/edycją
    aktywności (zadanie 12); wiersze bez nich po prostu nie mogą być
    sortowane po odległości.
- `backend/src/apps/activities/routes.py` — rozszerz `GET /api/activities`
  o opcjonalne `lat: float | None`, `lng: float | None` (muszą być
  podane razem — 422, jeśli ustawiono tylko jedno z nich):
  - Gdy oba są podane, oblicz odległość haversine w Pythonie między
    `(lat, lng)` a współrzędnymi `(latitude, longitude)` każdego wiersza,
    dołącz ją jako `distance_km` w odpowiedzi i posortuj rosnąco po
    odległości. Wiersze z pustymi współrzędnymi dostają
    `distance_km: null` i lądują na końcu (nie są wykluczane — rodzic
    powinien je nadal widzieć, tylko poniżej wyników z lokalizacją).
  - Łączy się ze wszystkimi istniejącymi filtrami (`indoor`,
    `max_duration_minutes`, `age_years`, `city`) operatorem `AND`;
    sortowanie po odległości działa tylko, gdy podano `lat`/`lng`, w
    przeciwnym razie kolejność sortowania pozostaje bez zmian (obecna
    domyślna, np. po nazwie lub `created_at`).
  - Umieść funkcję pomocniczą haversine w
    `backend/src/apps/activities/geo.py`, tak aby dało się ją testować
    jednostkowo w izolacji.
- Dodaj `backend/src/apps/activities/tests/test_geo.py` (pary o znanej
  odległości, np. dwie współrzędne oddalone o ~1 km) i rozszerz
  `test_routes.py` o zachowanie sortowania/obsługi pustych wartości oraz
  przypadek 422 dla „tylko jednego z lat/lng”.

#### Zmiany w danych

- `make make-migrations` — dwie nowe puste kolumny typu float.
- `make migrate`.

#### Zmiany we frontendzie

- `BrowsePage.tsx`: dodaj przycisk „Użyj mojej lokalizacji”. Po kliknięciu
  wywołaj `navigator.geolocation.getCurrentPosition`; w razie sukcesu
  zapisz `{lat, lng}` w stanie komponentu i dodaj je do stanu filtrów
  `useSearchParams` (`?lat=...&lng=...`), tak aby wyniki zostały
  ponownie posortowane; w razie odmowy/błędu pokaż przetłumaczony
  komunikat w treści strony i pozostaw sortowanie bez zmian (nie
  blokuj reszty strony).
  - To realny monit o uprawnienia urządzenia — utrzymuj go ściśle
    opcjonalnym, uruchamianym tylko przyciskiem; nigdy nie proś o
    lokalizację automatycznie przy ładowaniu strony.
- Pokaż `distance_km` (sformatowane, np. „2,3 km stąd”) na każdej karcie,
  gdy jest dostępne, po aktywowaniu lokalizacji.
- i18n: `filters.nearMe.button`, `filters.nearMe.denied`,
  `filters.nearMe.unsupported`, `activities.distanceAway` — wszystkie
  cztery pliki lokalizacji.

#### Poza zakresem

- Widok mapy — to zadanie dodaje tylko sortowanie listy, bez wizualnej
  mapy (patrz zadanie 13).
- Edycja współrzędnych z poziomu UI (zadanie 12 obsługuje wprowadzanie
  danych).
- Konfigurowalny promień odcięcia (np. „w promieniu 10 km”) — dla MVP
  tylko sortowanie.

#### Kryteria akceptacji

- Przy podanych `lat`/`lng` aktywności ze współrzędnymi są zwracane od
  najbliższych, z poprawnym `distance_km`; aktywności bez współrzędnych
  pojawiają się po wszystkich zlokalizowanych.
- Podanie tylko `lat` (bez `lng`) zwraca 422.
- Odmowa uprawnienia do lokalizacji w przeglądarce pozostawia stronę
  przeglądania używalną (brak błędu, brak zablokowanego UI), z pokazanym
  przetłumaczonym wyjaśnieniem.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
