---
status: todo
---

# 20. Mobile child profiles and personalized recommendations

## Summary

Port child profile management (task 10) and the "for my child" one-tap
age-filter shortcuts (task 11) to the `mobile/` app.

## Why

This closes out mobile parity with the web MVP feature set: the
personalization loop (save a child once, get one-tap filtered results and
anonymized-review context everywhere else) should work the same regardless
of which client a parent used to set it up.

## Backend changes

None — reuses `GET/POST/PATCH/DELETE /api/children` (task 10) and the
`child_id` param on `GET /api/activities` (task 11) exactly as already
defined.

## Data changes

None.

## Frontend changes

- New `ChildrenScreen`: list the signed-in user's children (name, computed
  age), add/edit/delete, using `@react-native-community/datetimepicker`
  (the native equivalent of the web's `@mantine/dates` `DateInput`) for
  birth date entry.
- Browse screen (task 18): render a row of chips — one per saved child,
  e.g. "For Mia (4y)" — above the filter controls when signed in with at
  least one saved child; tapping one sets the `child_id` query param the
  same way the web `BrowsePage.tsx` does, with a clear affordance to
  return to the manual age filter.
- A child saved/edited on mobile must be immediately usable for filtering
  on web (and vice versa) since both hit the same `/api/children` data —
  no client-side caching that could go stale between the two apps beyond
  a normal refetch-on-focus.
- i18n: reuse/extend the keys already defined for tasks 10/11 on web.

## Out of scope

- Any change to the recommendation logic itself (still a plain age-range
  match, per task 11's own out-of-scope note) — this task only ports the
  existing behavior to a new client.

## Acceptance criteria

- A child added on mobile appears correctly in the web app's "My
  Children" page and its age-filter chip, and vice versa.
- Selecting a child chip on mobile produces the same filtered result set
  as manually entering that child's current age.
- A user cannot access another user's children from the mobile app (same
  404/scoping behavior as web, since it's the same endpoint).

---

## Tłumaczenie (PL)

### 20. Profile dzieci i spersonalizowane rekomendacje na urządzeniach mobilnych

#### Podsumowanie

Przenieś zarządzanie profilami dzieci (zadanie 10) oraz jednoklikowe
skróty filtra wieku „dla mojego dziecka” (zadanie 11) do aplikacji
`mobile/`.

#### Dlaczego

To domyka parytet mobilny z zestawem funkcji MVP dla weba: pętla
personalizacji (zapisz dziecko raz, otrzymuj jednoklikowo przefiltrowane
wyniki i zanonimizowany kontekst w recenzjach wszędzie indziej) powinna
działać tak samo, niezależnie od tego, którego klienta rodzic użył do jej
skonfigurowania.

#### Zmiany w backendzie

Brak — wykorzystuje `GET/POST/PATCH/DELETE /api/children` (zadanie 10)
oraz parametr `child_id` w `GET /api/activities` (zadanie 11) dokładnie
tak, jak już zostały zdefiniowane.

#### Zmiany w danych

Brak.

#### Zmiany we frontendzie

- Nowy `ChildrenScreen`: lista dzieci zalogowanego użytkownika (imię,
  wyliczony wiek), dodawanie/edycja/usuwanie, z użyciem
  `@react-native-community/datetimepicker` (natywny odpowiednik
  webowego `DateInput` z `@mantine/dates`) do wprowadzania daty
  urodzenia.
- Ekran przeglądania (zadanie 18): renderuj rząd chipów — jeden na
  zapisane dziecko, np. „Dla Mii (4l)” — nad kontrolkami filtrów, gdy
  użytkownik jest zalogowany i ma co najmniej jedno zapisane dziecko;
  dotknięcie ustawia parametr zapytania `child_id` w taki sam sposób,
  jak robi to webowy `BrowsePage.tsx`, z wyraźną możliwością powrotu do
  ręcznego filtra wieku.
- Dziecko zapisane lub zmodyfikowane na urządzeniu mobilnym musi być od
  razu użyteczne do filtrowania w wersji webowej (i odwrotnie), ponieważ
  oba klienty korzystają z tych samych danych `/api/children` — bez
  cache’owania po stronie klienta, które mogłoby się zdezaktualizować
  między obiema aplikacjami, poza normalnym ponownym pobraniem po
  powrocie do ekranu.
- i18n: wykorzystaj/rozszerz klucze już zdefiniowane dla zadań 10/11 w
  wersji webowej.

#### Poza zakresem

- Jakakolwiek zmiana samej logiki rekomendacji (nadal zwykłe
  dopasowanie zakresu wieku, zgodnie z notatką „poza zakresem” w
  zadaniu 11) — to zadanie wyłącznie przenosi istniejące zachowanie do
  nowego klienta.

#### Kryteria akceptacji

- Dziecko dodane na urządzeniu mobilnym poprawnie pojawia się na
  webowej stronie „Moje dzieci” oraz w chipie filtra wieku, i odwrotnie.
- Wybranie chipa dziecka na urządzeniu mobilnym daje ten sam
  przefiltrowany zbiór wyników co ręczne wpisanie bieżącego wieku tego
  dziecka.
- Użytkownik nie może uzyskać dostępu do dzieci innego użytkownika z
  aplikacji mobilnej (to samo zachowanie 404/ograniczenia zakresu co w
  wersji webowej, ponieważ to ten sam endpoint).
