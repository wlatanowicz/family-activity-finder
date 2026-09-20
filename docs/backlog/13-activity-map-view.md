---
status: todo
---

# 13. Activity map view

## Summary

Add a map view to the browse page showing every filtered activity as a
marker at its coordinates, alongside the existing list view. This is the
"mapa z zaznaczonymi miejscami" idea from `docs/backlog/IDEAS.md` — a
literal map with marked places, not just the distance sort task 06 already
ships.

## Why

Task 06 added coordinates and "near me" sorting but explicitly left the map
itself out of scope ("this task only adds list sorting, no visual map").
Parents scanning "what's around here" often think spatially, and a map is
the most direct answer to that — it's also the only idea in `IDEAS.md` with
no existing task covering it at all.

## Backend changes

None — reuses `GET /api/activities` (including `latitude`/`longitude` from
task 06 and every filter from tasks 03–07); the map is a frontend
presentation of the same data the list view already fetches.

## Data changes

None.

## Frontend changes

- Add a map library to `frontend/package.json` — prefer `react-leaflet` +
  `leaflet` with OpenStreetMap tiles (no API key/billing required, unlike
  Google Maps or Mapbox — keeps this unblocked by any vendor account
  setup).
- `BrowsePage.tsx`: add a list/map view toggle (e.g. segmented control).
  Map view renders one marker per activity that has non-null
  `latitude`/`longitude` from the current filtered result set; activities
  without coordinates are omitted from the map (they still appear in list
  view) with a small translated note ("N activities without a location
  aren't shown on the map").
  - Marker click/tap opens a popup with name, category badge, and a "View
    details" link to the task 02 detail page.
  - Map recenters/refits bounds to the visible markers when filters change.
  - If "Use my location" (task 06) is active, show the user's own position
    as a distinct marker/icon and center the initial view on it; otherwise
    default the initial view to fit all markers (or a neutral world/region
    view when there are none).
- i18n: `browse.viewList`, `.viewMap`, `.mapMissingLocations` — all four
  locale files.

## Out of scope

- Drawing/selecting a search area on the map (e.g. "search this area") —
  filtering stays driven by the existing filter bar and location button,
  not map interaction, for MVP.
- Clustering markers at low zoom — acceptable to skip while activity
  volume is low post-launch; revisit if marker density makes the map
  unreadable.
- A map picker for entering coordinates in the admin form (task 12) —
  already explicitly out of scope there.

## Acceptance criteria

- Switching to map view shows one marker per filtered activity with
  coordinates, and applying a filter (e.g. task 03's indoor/outdoor)
  updates the markers shown.
- An activity with null coordinates never appears on the map but still
  appears in list view for the same filter state.
- Clicking a marker's popup link navigates to that activity's detail page.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 13. Widok mapy z aktywnościami

#### Podsumowanie

Dodaj do strony przeglądania widok mapy pokazujący każdą przefiltrowaną
aktywność jako znacznik w miejscu jej współrzędnych, obok istniejącego
widoku listy. To pomysł „mapa z zaznaczonymi miejscami” z
`docs/backlog/IDEAS.md` — dosłowna mapa z zaznaczonymi miejscami, a nie
tylko sortowanie po odległości, które dostarcza już zadanie 06.

#### Dlaczego

Zadanie 06 dodało współrzędne i sortowanie „w pobliżu”, ale wprost
wyłączyło samą mapę poza zakres („to zadanie dodaje tylko sortowanie
listy, bez wizualnej mapy”). Rodzice przeglądający „co jest w okolicy”
często myślą przestrzennie, a mapa to najbardziej bezpośrednia odpowiedź
na to — to też jedyny pomysł w `IDEAS.md`, który nie ma jeszcze żadnego
zadania.

#### Zmiany w backendzie

Brak — wykorzystuje `GET /api/activities` (łącznie z
`latitude`/`longitude` z zadania 06 oraz każdym filtrem z zadań 03–07);
mapa to prezentacja we frontendzie tych samych danych, które już pobiera
widok listy.

#### Zmiany w danych

Brak.

#### Zmiany we frontendzie

- Dodaj bibliotekę map do `frontend/package.json` — preferuj
  `react-leaflet` + `leaflet` z kafelkami OpenStreetMap (bez potrzeby
  klucza API/rozliczeń, w przeciwieństwie do Google Maps czy Mapbox —
  brak blokady w postaci konfiguracji konta u dostawcy).
- `BrowsePage.tsx`: dodaj przełącznik widoku lista/mapa (np.
  `SegmentedControl`). Widok mapy renderuje jeden znacznik na każdą
  aktywność z niepustym `latitude`/`longitude` z bieżącego przefiltrowanego
  zbioru wyników; aktywności bez współrzędnych są pomijane na mapie
  (nadal pojawiają się w widoku listy), z małą przetłumaczoną notatką
  („N aktywności bez lokalizacji nie jest pokazanych na mapie”).
  - Kliknięcie/dotknięcie znacznika otwiera dymek z nazwą, znacznikiem
    kategorii i linkiem „Zobacz szczegóły” do strony szczegółów z
    zadania 02.
  - Mapa ponownie centruje/dopasowuje granice widoku do widocznych
    znaczników przy zmianie filtrów.
  - Jeśli „Użyj mojej lokalizacji” (zadanie 06) jest aktywne, pokaż
    pozycję użytkownika jako odrębny znacznik/ikonę i wyśrodkuj na niej
    początkowy widok; w przeciwnym razie domyślnie dopasuj początkowy
    widok do wszystkich znaczników (lub neutralny widok świata/regionu,
    gdy ich brak).
- i18n: `browse.viewList`, `.viewMap`, `.mapMissingLocations` —
  wszystkie cztery pliki lokalizacji.

#### Poza zakresem

- Rysowanie/zaznaczanie obszaru wyszukiwania na mapie (np. „szukaj w tym
  obszarze”) — dla MVP filtrowanie pozostaje sterowane istniejącym
  paskiem filtrów i przyciskiem lokalizacji, a nie interakcją z mapą.
- Grupowanie znaczników przy małym przybliżeniu — akceptowalne do
  pominięcia, dopóki wolumen aktywności po starcie jest niski; wróć do
  tematu, jeśli gęstość znaczników uczyni mapę nieczytelną.
- Wybieranie współrzędnych na mapie w formularzu administracyjnym
  (zadanie 12) — tam już wprost poza zakresem.

#### Kryteria akceptacji

- Przełączenie na widok mapy pokazuje jeden znacznik na każdą
  przefiltrowaną aktywność ze współrzędnymi, a zastosowanie filtra (np.
  wnętrze/na zewnątrz z zadania 03) aktualizuje pokazywane znaczniki.
- Aktywność z pustymi współrzędnymi nigdy nie pojawia się na mapie, ale
  nadal pojawia się w widoku listy dla tego samego stanu filtrów.
- Kliknięcie linku w dymku znacznika nawiguje do strony szczegółów tej
  aktywności.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
