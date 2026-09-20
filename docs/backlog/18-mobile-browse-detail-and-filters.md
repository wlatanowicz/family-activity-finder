---
status: todo
---

# 18. Mobile browse, detail, and filters

## Summary

Port the web browse page, activity detail page, and every filter (indoor/
outdoor, duration, age, city, "near me," weather) to native screens in the
`mobile/` app from task 17, against the exact same `GET /api/activities`
endpoint and query parameters tasks 01–07 already define.

## Why

Browse + filter is the core "help parents discover activities" experience
the whole product is built around (tasks 01–07); it has to exist on mobile
for the platform to be more than an auth shell, and it needs no new backend
work since the API is already client-agnostic.

## Backend changes

None — reuses `GET /api/activities` and `GET /api/activities/{id}` exactly
as tasks 01–07 defined them for the web frontend.

## Data changes

None.

## Frontend changes

- New `BrowseScreen` in `mobile/`: fetches `GET /api/activities`, renders a
  scrollable list of activity cards (name, category badge, address,
  truncated description — same fields as the web `BrowsePage.tsx`).
- New `ActivityDetailScreen`, navigated to on card tap, fetches
  `GET /api/activities/{id}` (task 02's endpoint).
- Filter UI native equivalents for every task 03–07 filter, all
  contributing to the same query params the web app builds
  (`is_indoor`, `max_duration_minutes`, `age_years`, `city`, `lat`/`lng`,
  weather tag): a filter sheet/modal is a more natural native pattern than
  the web's inline filter bar — use whichever native pattern (bottom
  sheet, dedicated filter screen) fits the chosen navigation library from
  task 17, as long as the resulting query params match.
- "Near me" (task 06): use `expo-location`'s permission request +
  `getCurrentPositionAsync` in place of the browser Geolocation API;
  same opt-in-only rule — never request location on screen mount, only on
  an explicit "Use my location" tap.
- Rainy-day shortcut (task 07): a one-tap button applying the weather
  filter's preset value, same as the web version.
- i18n: reuse/extend the same translation keys ported in task 17.

## Out of scope

- The map view from task 13 — `react-native-maps` needs native module
  linking and an EAS development build rather than Expo Go, which is a
  meaningfully bigger lift than the list-based screens here. Track it as
  its own follow-up task if a mobile map is wanted later; this task ships
  list-view browsing only.
- Favorites, ratings/reviews, child profiles — tasks 19–20.

## Acceptance criteria

- The mobile browse screen and the web browse page return/display the
  same activities for equivalent filter selections (same query params
  against the same backend).
- Tapping an activity opens its detail screen with the same data the web
  detail page shows.
- The "near me" filter correctly handles a denied location permission
  without crashing the screen (mirrors task 06's web acceptance criterion).
- Every filter from tasks 03–07 is reachable and combinable on mobile.

---

## Tłumaczenie (PL)

### 18. Przeglądanie, szczegóły i filtry na urządzeniach mobilnych

#### Podsumowanie

Przenieś webową stronę przeglądania, stronę szczegółów aktywności i
każdy filtr (wnętrze/na zewnątrz, czas trwania, wiek, miasto, „w
pobliżu”, pogoda) na ekrany natywne w aplikacji `mobile/` z zadania 17,
względem dokładnie tego samego endpointu `GET /api/activities` i tych
samych parametrów zapytania, jakie definiują już zadania 01–07.

#### Dlaczego

Przeglądanie + filtry to podstawowe doświadczenie „pomóż rodzicom
odkrywać aktywności”, wokół którego zbudowany jest cały produkt (zadania
01–07); musi istnieć na urządzeniach mobilnych, aby platforma była czymś
więcej niż powłoką autoryzacji, i nie wymaga żadnej nowej pracy po
stronie backendu, ponieważ API jest już niezależne od klienta.

#### Zmiany w backendzie

Brak — wykorzystuje `GET /api/activities` i
`GET /api/activities/{id}` dokładnie tak, jak zdefiniowały je zadania
01–07 dla aplikacji webowej.

#### Zmiany w danych

Brak.

#### Zmiany we frontendzie

- Nowy `BrowseScreen` w `mobile/`: pobiera `GET /api/activities`,
  renderuje przewijaną listę kart aktywności (nazwa, znacznik kategorii,
  adres, skrócony opis — te same pola co w webowym `BrowsePage.tsx`).
- Nowy `ActivityDetailScreen`, otwierany po dotknięciu karty, pobiera
  `GET /api/activities/{id}` (endpoint z zadania 02).
- Natywne odpowiedniki UI dla każdego filtra z zadań 03–07, wszystkie
  zasilające te same parametry zapytania, jakie buduje aplikacja webowa
  (`is_indoor`, `max_duration_minutes`, `age_years`, `city`,
  `lat`/`lng`, znacznik pogody): arkusz/modal filtrów to bardziej
  naturalny wzorzec mobilny niż webowy pasek filtrów w linii — użyj
  dowolnego natywnego wzorca (bottom sheet, dedykowany ekran filtrów),
  który pasuje do wybranej w zadaniu 17 biblioteki nawigacji, dopóki
  wynikowe parametry zapytania się zgadzają.
- „W pobliżu” (zadanie 06): użyj żądania uprawnień z `expo-location` +
  `getCurrentPositionAsync` zamiast API geolokalizacji przeglądarki; ta
  sama zasada — wyłącznie na wyraźne żądanie, nigdy nie proś o
  lokalizację automatycznie przy montowaniu ekranu.
- Skrót „dzień deszczowy” (zadanie 07): przycisk jednoklikowy stosujący
  ustawioną z góry wartość filtra pogody, tak samo jak w wersji webowej.
- i18n: wykorzystaj/rozszerz te same klucze tłumaczeń przeniesione w
  zadaniu 17.

#### Poza zakresem

- Widok mapy z zadania 13 — `react-native-maps` wymaga linkowania
  modułów natywnych i deweloperskiego builda EAS zamiast Expo Go, co
  jest zauważalnie większym nakładem niż ekrany oparte na listach tutaj.
  Traktuj to jako osobne zadanie kontynuacyjne, jeśli mobilna mapa
  będzie kiedyś potrzebna; to zadanie dostarcza wyłącznie przeglądanie
  w widoku listy.
- Ulubione, oceny/recenzje, profile dzieci — zadania 19–20.

#### Kryteria akceptacji

- Mobilny ekran przeglądania i webowa strona przeglądania
  zwracają/wyświetlają te same aktywności dla równoważnych wyborów
  filtrów (te same parametry zapytania względem tego samego backendu).
- Dotknięcie aktywności otwiera jej ekran szczegółów z tymi samymi
  danymi, jakie pokazuje webowa strona szczegółów.
- Filtr „w pobliżu” poprawnie obsługuje odmowę uprawnienia lokalizacji
  bez awarii ekranu (odzwierciedla kryterium akceptacji zadania 06 dla
  weba).
- Każdy filtr z zadań 03–07 jest dostępny i można go łączyć na
  urządzeniach mobilnych.
