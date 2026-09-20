---
status: todo
---

# 19. Mobile favorites and ratings

## Summary

Port favorites (task 08) and community ratings/reviews — including photo
attachments (task 15) and the optional anonymized child context (task 16)
— to the `mobile/` app, against the same endpoints the web frontend uses.

## Why

These are the engagement/trust-loop features (favorites, ratings, photos,
child context) already scoped and built for web; shipping them on mobile
is what makes the native app a full alternative to the web app rather than
a read-only browse shell.

## Backend changes

None — reuses `POST/DELETE /api/favorites`, `GET/POST/DELETE
/api/activities/{id}/ratings[...]`, the photo upload/delete endpoints from
task 15, and the `child_id`/`child_age_years` fields from task 16 exactly
as already defined for the web frontend.

## Data changes

None.

## Frontend changes

- Favorites (task 08): a save/heart button on activity cards and the
  detail screen, calling the same `POST`/`DELETE /api/favorites`
  endpoints; signed-out tap shows a native prompt to sign in instead of
  hiding the button. A `FavoritesScreen` listing the signed-in user's
  saved activities.
- Ratings/reviews (task 09): star-input + optional comment on the detail
  screen, submitting to the same `POST .../ratings` endpoint; a reviews
  list below showing score, comment, masked identity, relative date, same
  privacy rules as web (no full email ever shown).
- Photo attachments (task 15): use `expo-image-picker` (camera or library)
  to let a user attach up to the backend's per-rating cap, uploading via
  the same multipart endpoint; render each review's photos as a
  thumbnail strip with a full-screen viewer on tap.
- Anonymized child context (task 16): an optional "Reviewing for" picker
  (native equivalent of the web select) listing the user's saved children
  by name — shown only to the reviewer while filling out their own
  review — submitting the chosen `child_id`; reviews list renders
  `child_age_years` when present (e.g. "Parent of a 4-year-old"), never a
  name, exactly as the API already guarantees.
- i18n: reuse/extend the keys already defined for tasks 08/09/15/16 on
  web.

## Out of scope

- Anything not already in scope for tasks 08/09/15/16 on web (e.g.
  moderation, per-child favorites) — this task is a straight port, not a
  chance to expand scope.

## Acceptance criteria

- Favoriting/unfavoriting on mobile is reflected in `GET /api/favorites`
  identically to the web flow (same account, same result set).
- A rating submitted from mobile — with an attached photo and an optional
  linked child — appears correctly in the web app's reviews list (average
  score, photo thumbnail, anonymized child age line), confirming both
  clients share one backend and one data model.
- Signed-out users can read ratings/photos but are prompted to sign in
  before submitting one, matching the web behavior.

---

## Tłumaczenie (PL)

### 19. Ulubione i oceny na urządzeniach mobilnych

#### Podsumowanie

Przenieś ulubione (zadanie 08) oraz oceny/recenzje społeczności — łącznie
z załącznikami zdjęciowymi (zadanie 15) i opcjonalnym zanonimizowanym
kontekstem dziecka (zadanie 16) — do aplikacji `mobile/`, względem tych
samych endpointów, jakich używa aplikacja webowa.

#### Dlaczego

To pętla angażująca/budująca zaufanie (ulubione, oceny, zdjęcia, kontekst
dziecka), już zaplanowana i zbudowana dla weba; dostarczenie jej na
urządzeniach mobilnych sprawia, że aplikacja natywna staje się pełną
alternatywą dla aplikacji webowej, a nie powłoką tylko do przeglądania.

#### Zmiany w backendzie

Brak — wykorzystuje `POST/DELETE /api/favorites`, `GET/POST/DELETE
/api/activities/{id}/ratings[...]`, endpointy przesyłania/usuwania
zdjęć z zadania 15 oraz pola `child_id`/`child_age_years` z zadania 16
dokładnie tak, jak zostały już zdefiniowane dla aplikacji webowej.

#### Zmiany w danych

Brak.

#### Zmiany we frontendzie

- Ulubione (zadanie 08): przycisk zapisu/serca na kartach aktywności i
  ekranie szczegółów, wywołujący te same endpointy `POST`/`DELETE
  /api/favorites`; dotknięcie przez niezalogowanego pokazuje natywny
  monit o zalogowanie zamiast ukrywania przycisku. `FavoritesScreen`
  wyświetlający zapisane aktywności zalogowanego użytkownika.
- Oceny/recenzje (zadanie 09): wprowadzanie gwiazdek + opcjonalny
  komentarz na ekranie szczegółów, wysyłane do tego samego endpointu
  `POST .../ratings`; lista recenzji poniżej pokazująca ocenę,
  komentarz, zamaskowaną tożsamość, względną datę — te same zasady
  prywatności co w wersji webowej (nigdy nie pokazuj pełnego adresu
  e-mail).
- Załączniki zdjęciowe (zadanie 15): użyj `expo-image-picker` (aparat
  lub biblioteka zdjęć), aby pozwolić użytkownikowi dołączyć zdjęcia do
  limitu backendu na ocenę, przesyłane tym samym endpointem
  multipart; renderuj zdjęcia każdej recenzji jako pasek miniatur z
  podglądem pełnoekranowym po dotknięciu.
- Zanonimizowany kontekst dziecka (zadanie 16): opcjonalny wybór
  „Recenzja dotyczy” (natywny odpowiednik webowego selecta) z listą
  zapisanych dzieci użytkownika po imieniu — pokazywany wyłącznie
  recenzentowi podczas wypełniania własnej recenzji — wysyłający
  wybrane `child_id`; lista recenzji renderuje `child_age_years`, gdy
  obecne (np. „Rodzic 4-latka”), nigdy imię, dokładnie tak, jak
  gwarantuje już API.
- i18n: wykorzystaj/rozszerz klucze już zdefiniowane dla zadań
  08/09/15/16 w wersji webowej.

#### Poza zakresem

- Wszystko, co nie jest już w zakresie zadań 08/09/15/16 dla weba (np.
  moderacja, ulubione per dziecko) — to zadanie jest bezpośrednim
  przeniesieniem, a nie okazją do rozszerzenia zakresu.

#### Kryteria akceptacji

- Dodanie/usunięcie z ulubionych na urządzeniu mobilnym odzwierciedla
  się identycznie w `GET /api/favorites` co w przepływie webowym (to
  samo konto, ten sam zbiór wyników).
- Ocena wysłana z aplikacji mobilnej — z dołączonym zdjęciem i
  opcjonalnie powiązanym dzieckiem — poprawnie pojawia się na liście
  recenzji w aplikacji webowej (średnia ocena, miniatura zdjęcia,
  zanonimizowana linia wieku dziecka), potwierdzając, że oba klienty
  współdzielą jeden backend i jeden model danych.
- Niezalogowani użytkownicy mogą czytać oceny/zdjęcia, ale są proszeni o
  zalogowanie przed wysłaniem własnej oceny, zgodnie z zachowaniem
  weba.
