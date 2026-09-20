---
status: todo
---

# 02. Client-side routing and activity detail page

## Summary

Introduce client-side routing (the app currently has none — it's a single
`App.tsx` view) and use it to ship the first navigable page: an activity
detail view reachable by clicking an activity on the browse page, at a
shareable URL.

## Why

Every task from here on (favorites page, ratings on a specific activity,
children management, admin forms) needs more than one screen. Routing is
infrastructure with no user value on its own, so it ships bundled with the
first page that actually needs it rather than as a standalone task.

## Backend changes

- Add `GET /api/activities/{activity_id}` to
  `backend/src/apps/activities/routes.py`:
  - `activity_id: UUID` path param.
  - 404 with `ApiErrorCode.activity_not_found` (added in task 01) when no
    row matches.
  - 503 with `ApiErrorCode.database_not_configured` when `session is None`
    (same convention as the list endpoint and `users` routes).
  - Response: full `Activity` fields (`id`, `name`, `description`,
    `address`, `category`, `created_at`).
- Add a test in a new `backend/src/apps/activities/tests/test_routes.py`
  covering: 404 for unknown id, 200 with the right shape for an existing
  row, and 503 without a database.

## Frontend changes

- Add `react-router-dom` (`^7`) to `frontend/package.json`.
- Wrap the app in a `BrowserRouter` in `frontend/src/main.tsx`.
- Split `App.tsx`'s browse markup into a `pages/BrowsePage.tsx` (list +
  fetch from task 01) and add `pages/ActivityDetailPage.tsx`:
  - Route `/` → `BrowsePage`, `/activities/:activityId` → `ActivityDetailPage`.
  - Keep the header (title, language selector, auth panel/status) in a
    shared `Layout` component rendered around the routed pages via
    `<Outlet />`, so sign-in state persists across navigation.
  - `ActivityDetailPage` fetches `GET /api/activities/:activityId`, shows
    name, category, address, full description; shows a "not found" message
    (translated) on 404; each `BrowsePage` card links to
    `/activities/${id}` with a router `Link`.
- i18n: add `activities.detail.notFound`, `activities.detail.backToBrowse`
  to `en.json` and the other three locale files.

## Data changes

None — reuses the `activities` table from task 01.

## Out of scope

- Deep-linkable filters (query-string state) — introduced incrementally as
  filters are added in tasks 03–07.
- Favorites/ratings on the detail page (tasks 08–09 add to this page).

## Acceptance criteria

- Navigating to `/activities/<real-id>` directly (e.g. page refresh, pasted
  link) renders the detail page without going through the browse page.
- Navigating to `/activities/<unknown-id>` shows a translated "not found"
  state, not a crash.
- Clicking an activity card on the browse page navigates without a full
  page reload.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 02. Routing po stronie klienta i strona szczegółów aktywności

#### Podsumowanie

Wprowadź routing po stronie klienta (aplikacja obecnie go nie ma — to
pojedynczy widok `App.tsx`) i wykorzystaj go do dostarczenia pierwszej
strony z nawigacją: widoku szczegółów aktywności, dostępnego po kliknięciu
aktywności na stronie przeglądania, pod adresem URL, którym można się
podzielić.

#### Dlaczego

Każde kolejne zadanie (strona ulubionych, oceny konkretnej aktywności,
zarządzanie dziećmi, formularze administracyjne) wymaga więcej niż jednego
ekranu. Routing to infrastruktura bez samodzielnej wartości dla
użytkownika, więc dostarczamy go razem z pierwszą stroną, która faktycznie
go potrzebuje, zamiast jako osobne zadanie.

#### Zmiany w backendzie

- Dodaj `GET /api/activities/{activity_id}` do
  `backend/src/apps/activities/routes.py`:
  - Parametr ścieżki `activity_id: UUID`.
  - 404 z `ApiErrorCode.activity_not_found` (dodanym w zadaniu 01), gdy
    żaden wiersz nie pasuje.
  - 503 z `ApiErrorCode.database_not_configured`, gdy `session is None`
    (ta sama konwencja co w endpoincie listy i w trasach `users`).
  - Odpowiedź: pełne pola `Activity` (`id`, `name`, `description`,
    `address`, `category`, `created_at`).
- Dodaj test w nowym pliku
  `backend/src/apps/activities/tests/test_routes.py`, obejmujący: 404 dla
  nieznanego id, 200 z prawidłowym kształtem dla istniejącego wiersza oraz
  503 bez bazy danych.

#### Zmiany we frontendzie

- Dodaj `react-router-dom` (`^7`) do `frontend/package.json`.
- Opakuj aplikację w `BrowserRouter` w `frontend/src/main.tsx`.
- Wydziel znacznik przeglądania z `App.tsx` do `pages/BrowsePage.tsx`
  (lista + pobieranie danych z zadania 01) i dodaj
  `pages/ActivityDetailPage.tsx`:
  - Trasa `/` → `BrowsePage`, `/activities/:activityId` →
    `ActivityDetailPage`.
  - Zachowaj nagłówek (tytuł, wybór języka, panel/status logowania) we
    wspólnym komponencie `Layout`, renderowanym wokół stron obsługiwanych
    przez router za pomocą `<Outlet />`, tak aby stan zalogowania
    przetrwał nawigację.
  - `ActivityDetailPage` pobiera dane z `GET /api/activities/:activityId`,
    pokazuje nazwę, kategorię, adres i pełny opis; przy 404 pokazuje
    przetłumaczony komunikat „nie znaleziono”; każda karta w `BrowsePage`
    prowadzi do `/activities/${id}` za pomocą `Link` z routera.
- i18n: dodaj `activities.detail.notFound`, `activities.detail.backToBrowse`
  do `en.json` oraz pozostałych trzech plików lokalizacji.

#### Zmiany w danych

Brak — wykorzystuje tabelę `activities` z zadania 01.

#### Poza zakresem

- Filtry z linkami bezpośrednimi (stan w query stringu) — wprowadzane
  stopniowo wraz z dodawaniem filtrów w zadaniach 03–07.
- Ulubione/oceny na stronie szczegółów (zadania 08–09 dodają je do tej
  strony).

#### Kryteria akceptacji

- Bezpośrednie przejście pod `/activities/<real-id>` (np. odświeżenie
  strony, wklejony link) renderuje stronę szczegółów bez przechodzenia
  przez stronę przeglądania.
- Przejście pod `/activities/<unknown-id>` pokazuje przetłumaczony stan
  „nie znaleziono”, a nie błąd aplikacji.
- Kliknięcie karty aktywności na stronie przeglądania nawiguje bez
  pełnego przeładowania strony.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
