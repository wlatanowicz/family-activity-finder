---
status: todo
---

# 07. Weather-suitability filter and rainy-day shortcut

## Summary

Add a weather-suitability tag to each activity and a filter for it,
completing the discovery summary's MVP filter set ("weather"). Ship a
one-tap "Rainy day?" shortcut on the browse page — pulled forward from the
"Future Features" list because it turns out to be nothing more than this
filter with a preset value, not a separate system (no real weather-API
integration required for MVP).

## Why

Parents most often reach for a weather filter reactively — "it's raining,
what can we do today" — so the UI framing (a shortcut button) matters as
much as the underlying field.

## Backend changes

- `backend/src/apps/activities/models.py` — add:
  - `WeatherSuitability` `StrEnum`: `any`, `sunny_only`, `rainy_friendly`.
    Use the existing `to_sql_enum` helper (`src/utils/db.py`) the same way
    `UserStatus`/`AuthProvider` do, since this is a small, stable, closed
    set (unlike `category` in task 01).
  - `Activity.weather_suitability: WeatherSuitability` — not nullable,
    default `WeatherSuitability.any`.
- `backend/src/apps/activities/routes.py` — extend `GET /api/activities`
  with optional query param `weather: Literal["sunny", "rainy"] | None`:
  - `weather=rainy` → include rows where `weather_suitability` is `any` or
    `rainy_friendly`.
  - `weather=sunny` → include rows where `weather_suitability` is `any` or
    `sunny_only`.
  - Omitted → no filtering (current behavior). Combines with `AND`
    alongside all filters from tasks 03–06.
- Extend `test_routes.py` with cases for each `weather` value and the
  default (`any`) always matching.

## Data changes

- `make make-migrations` — new enum type + column,
  `server_default='any'` so existing rows backfill cleanly.
- `make migrate`.

## Frontend changes

- `BrowsePage.tsx`: add a small row of quick-filter chips above/alongside
  the existing filter bar: "☀️ Sunny day", "🌧️ Rainy day", and a way to
  clear back to "Any weather" — wired into the same `useSearchParams`
  filter state (`?weather=rainy`).
  - The "🌧️ Rainy day" chip is the "rainy-day suggestions" feature: no
    separate page or logic, just this filter value plus (for extra
    visibility) auto-combining it with `indoor=true` from task 03 when
    the user taps it specifically labeled as a shortcut — i.e. tapping
    "Rainy day" sets both `weather=rainy` and `indoor=true` in one click,
    since an indoor activity is what "rainy day" actually means to a
    parent. Sunny-day chip does not force `indoor=false` (outdoor-only
    would be too restrictive on a nice day).
- Show a small weather-suitability icon/badge on cards where
  `weather_suitability != any`.
- i18n: `filters.weather.sunny`, `.rainy`, `.any` — all four locale files.

## Out of scope

- Real weather-API integration (auto-detecting today's actual weather) —
  explicitly deferred; this task is a manually-selected filter only.
- Hourly/forecast-based suggestions.

## Acceptance criteria

- `GET /api/activities?weather=rainy` includes `any` and `rainy_friendly`
  rows, excludes `sunny_only`.
- Tapping the "Rainy day" chip sets both weather and indoor filters and the
  URL reflects both; the list narrows accordingly.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 07. Filtr dopasowania do pogody i skrót „dzień deszczowy”

#### Podsumowanie

Dodaj znacznik dopasowania do pogody dla każdej aktywności i filtr dla
niego, uzupełniając zestaw filtrów MVP z podsumowania odkrycia
(„pogoda”). Dostarcz jednoklikowy skrót „Dzień deszczowy?” na stronie
przeglądania — przeniesiony z listy „Przyszłych funkcji”, ponieważ okazuje
się być niczym więcej niż tym filtrem z ustawioną z góry wartością, a nie
osobnym systemem (bez potrzeby prawdziwej integracji z API pogodowym w
MVP).

#### Dlaczego

Rodzice najczęściej sięgają po filtr pogody reaktywnie — „pada deszcz, co
możemy dziś robić” — więc oprawa UI (przycisk-skrót) jest tak samo ważna
jak samo pole w danych.

#### Zmiany w backendzie

- `backend/src/apps/activities/models.py` — dodaj:
  - `StrEnum` `WeatherSuitability`: `any`, `sunny_only`, `rainy_friendly`.
    Użyj istniejącego helpera `to_sql_enum` (`src/utils/db.py`) w taki
    sam sposób, jak robią to `UserStatus`/`AuthProvider`, ponieważ to
    mały, stabilny, zamknięty zbiór (w przeciwieństwie do `category` z
    zadania 01).
  - `Activity.weather_suitability: WeatherSuitability` — niepuste,
    domyślnie `WeatherSuitability.any`.
- `backend/src/apps/activities/routes.py` — rozszerz `GET /api/activities`
  o opcjonalny parametr zapytania `weather: Literal["sunny", "rainy"] |
  None`:
  - `weather=rainy` → dołącz wiersze, gdzie `weather_suitability` to
    `any` lub `rainy_friendly`.
  - `weather=sunny` → dołącz wiersze, gdzie `weather_suitability` to
    `any` lub `sunny_only`.
  - Pominięte → brak filtrowania (bieżące zachowanie). Łączy się
    operatorem `AND` z filtrami z zadań 03–06.
- Rozszerz `test_routes.py` o przypadki dla każdej wartości `weather`
  oraz domyślnej (`any`), która zawsze pasuje.

#### Zmiany w danych

- `make make-migrations` — nowy typ enum + kolumna,
  `server_default='any'`, tak aby istniejące wiersze zostały poprawnie
  uzupełnione.
- `make migrate`.

#### Zmiany we frontendzie

- `BrowsePage.tsx`: dodaj mały rząd chipów szybkiego filtrowania
  nad/obok istniejącego paska filtrów: „☀️ Słoneczny dzień”, „🌧️
  Deszczowy dzień” oraz sposób powrotu do „Dowolna pogoda” — podpięte do
  tego samego stanu filtrów `useSearchParams` (`?weather=rainy`).
  - Chip „🌧️ Deszczowy dzień” to funkcja „sugestii na deszczowy dzień”:
    bez osobnej strony czy logiki, tylko ta wartość filtra plus (dla
    dodatkowej widoczności) automatyczne połączenie z `indoor=true` z
    zadania 03, gdy użytkownik dotknie chipa opisanego właśnie jako
    skrót — czyli dotknięcie „Dzień deszczowy” ustawia jednym kliknięciem
    zarówno `weather=rainy`, jak i `indoor=true`, ponieważ aktywność w
    pomieszczeniu to właśnie to, co „dzień deszczowy” oznacza dla
    rodzica. Chip „dzień słoneczny” nie wymusza `indoor=false`
    (ograniczenie tylko do aktywności na zewnątrz byłoby zbyt
    restrykcyjne w ładny dzień).
- Pokaż mały znacznik/ikonę dopasowania do pogody na kartach, gdzie
  `weather_suitability != any`.
- i18n: `filters.weather.sunny`, `.rainy`, `.any` — wszystkie cztery
  pliki lokalizacji.

#### Poza zakresem

- Prawdziwa integracja z API pogodowym (automatyczne wykrywanie
  dzisiejszej pogody) — wyraźnie odłożone; to zadanie to wyłącznie
  ręcznie wybierany filtr.
- Sugestie oparte na prognozie godzinowej.

#### Kryteria akceptacji

- `GET /api/activities?weather=rainy` obejmuje wiersze `any` i
  `rainy_friendly`, wyklucza `sunny_only`.
- Dotknięcie chipa „Dzień deszczowy” ustawia zarówno filtr pogody, jak i
  filtr wnętrza, a URL odzwierciedla oba; lista odpowiednio się zawęża.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
