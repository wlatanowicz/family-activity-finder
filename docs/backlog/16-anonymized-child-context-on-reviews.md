---
status: todo
---

# 16. Anonymized child context on reviews

## Summary

Let a parent optionally attach one of their saved child profiles (task 10)
to a rating/review (task 09) when submitting it, and show that context to
other readers in fully anonymized form (e.g. "Parent of a 4-year-old"
instead of any name or identity). This closes the remaining half of the
"profil dziecka... uzywany przy filtrowaniu i dodawaniu opinii... uzywamy
zaononimizowanych danych przy opiniach" idea from `docs/backlog/IDEAS.md` —
the filtering half already shipped as task 11; the review-linking half
never had a task.

## Why

`IDEAS.md` describes the child profile as feeding two things: filtering
(task 11) and reviews, with an explicit privacy constraint that the profile
itself stays private and only anonymized data (the child's age, never name
or identity) surfaces in reviews. Today task 09's reviews only show a
masked identity for the *account*, with no link to *which child* the review
is actually about — a detail other parents reading a review would find
useful ("this was written by a parent of a 4-year-old" is more relevant
context than a masked email).

## Backend changes

- `backend/src/apps/ratings/models.py` — add to `Rating`:
  - `child_id: UUID | None` — `foreign_key="child_profiles.id"`, nullable
    (attaching a child is optional; a rating with none behaves exactly as
    task 09 already specifies).
- `backend/src/apps/ratings/routes.py`:
  - `POST /api/activities/{activity_id}/ratings` (task 09): extend the
    request body with optional `child_id: UUID | None`. If supplied,
    verify it belongs to the calling user (404 `child_not_found` if not —
    same non-leaking pattern task 10 uses) and store it on the rating.
  - `GET /api/activities/{activity_id}/ratings` (task 09): when a rating
    has a `child_id`, compute the linked child's current `age_years` (same
    computation task 10's `GET /api/children` uses) server-side and
    include it as `child_age_years: int | None` in the per-rating response
    — **never** include the child's name, id, or the owning user's
    identity beyond what task 09 already exposes. This is the anonymized
    data `IDEAS.md` calls for: an age number, nothing that could identify
    the specific child or account.
- Tests: rating with a `child_id` returns the correct `child_age_years` in
  the public listing without leaking the child's name/id; a `child_id`
  belonging to another user is rejected (404, not silently accepted or
  ignored); rating with no `child_id` behaves exactly as before this task.

## Data changes

- `make make-migrations` — new nullable `child_id` FK column on `ratings`.
- `make migrate`.

## Frontend changes

- `ActivityDetailPage.tsx`'s rating submission widget (task 09): when
  signed in with at least one saved child (task 10), add an optional
  "Reviewing for" select listing the user's children by name (name is only
  ever shown to the reviewer themselves, filling out their own form — never
  sent anywhere but this request) with a "Prefer not to say" default;
  submitting includes the chosen `child_id`.
- Reviews list: render `child_age_years` when present as a small
  translated line under the review, e.g. "Parent of a 4-year-old" — never
  render a child's name or any other identifying detail, since none is
  ever returned by the API for this purpose.
- i18n: `ratings.reviewingFor.label`, `.notSaid`,
  `ratings.childContext` (`"Parent of a {{age}}-year-old"`) — all four
  locale files.

## Out of scope

- Attaching more than one child to a single rating — one optional child
  per rating, matching the "For Mia" single-child selection pattern task
  11 already established.
- Any use of this link for recommendation/personalization beyond display —
  that would extend task 11's scope, not this one.

## Acceptance criteria

- A rating submitted with a `child_id` shows the child's current age (not
  name, not id) on the public reviews list; a rating submitted without one
  shows no child context line, matching task 09's current behavior.
- A user cannot attach another user's child to their rating (404, no data
  leak).
- Nothing in the public API response for ratings ever includes a child's
  name or id — only the derived age.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 16. Zanonimizowany kontekst dziecka w recenzjach

#### Podsumowanie

Pozwól rodzicowi opcjonalnie dołączyć jeden z zapisanych profili dzieci
(zadanie 10) do oceny/recenzji (zadanie 09) podczas jej wysyłania i pokaż
ten kontekst innym czytelnikom w pełni zanonimizowanej formie (np.
„Rodzic 4-latka” zamiast jakiegokolwiek imienia czy tożsamości). To
zamyka pozostałą połowę pomysłu „profil dziecka... używany przy
filtrowaniu i dodawaniu opinii... używamy zanonimizowanych danych przy
opiniach” z `docs/backlog/IDEAS.md` — połowa dotycząca filtrowania
została już dostarczona jako zadanie 11; połowa dotycząca powiązania z
recenzjami nigdy nie miała własnego zadania.

#### Dlaczego

`IDEAS.md` opisuje profil dziecka jako zasilający dwie rzeczy:
filtrowanie (zadanie 11) i recenzje, z wyraźnym ograniczeniem
prywatności, że sam profil pozostaje prywatny, a w recenzjach pojawiają
się wyłącznie zanonimizowane dane (wiek dziecka, nigdy imię czy
tożsamość). Obecnie recenzje z zadania 09 pokazują wyłącznie
zamaskowaną tożsamość *konta*, bez powiązania z tym, *którego dziecka*
recenzja faktycznie dotyczy — szczegół, który inni rodzice czytający
recenzję uznaliby za przydatny („to napisał rodzic 4-latka” to bardziej
istotny kontekst niż zamaskowany e-mail).

#### Zmiany w backendzie

- `backend/src/apps/ratings/models.py` — dodaj do `Rating`:
  - `child_id: UUID | None` — `foreign_key="child_profiles.id"`, puste
    (dołączenie dziecka jest opcjonalne; ocena bez tego pola zachowuje
    się dokładnie tak, jak już opisuje zadanie 09).
- `backend/src/apps/ratings/routes.py`:
  - `POST /api/activities/{activity_id}/ratings` (zadanie 09): rozszerz
    treść żądania o opcjonalne `child_id: UUID | None`. Jeśli podane,
    zweryfikuj, że należy do wywołującego użytkownika (404
    `child_not_found`, jeśli nie — ten sam wzorzec nie-ujawniania, jakiego
    używa zadanie 10) i zapisz je przy ocenie.
  - `GET /api/activities/{activity_id}/ratings` (zadanie 09): gdy ocena
    ma `child_id`, wylicz bieżący `age_years` powiązanego dziecka (to
    samo wyliczenie, którego używa `GET /api/children` z zadania 10) po
    stronie serwera i dołącz je jako `child_age_years: int | None` w
    odpowiedzi dla każdej oceny — **nigdy** nie dołączaj imienia dziecka,
    jego id ani tożsamości właściciela poza tym, co już ujawnia zadanie
    09. To zanonimizowane dane, o które prosi `IDEAS.md`: liczba
    oznaczająca wiek, nic, co mogłoby zidentyfikować konkretne dziecko
    lub konto.
- Testy: ocena z `child_id` zwraca poprawny `child_age_years` na
  publicznej liście bez ujawniania imienia/id dziecka; `child_id`
  należące do innego użytkownika jest odrzucane (404, nie ciche
  zaakceptowanie ani zignorowanie); ocena bez `child_id` zachowuje się
  dokładnie tak jak przed tym zadaniem.

#### Zmiany w danych

- `make make-migrations` — nowa pusta kolumna FK `child_id` w `ratings`.
- `make migrate`.

#### Zmiany we frontendzie

- Widget wysyłania oceny w `ActivityDetailPage.tsx` (zadanie 09): gdy
  użytkownik jest zalogowany i ma co najmniej jedno zapisane dziecko
  (zadanie 10), dodaj opcjonalny wybór „Recenzja dotyczy” z listą dzieci
  użytkownika po imieniu (imię jest pokazywane wyłącznie recenzentowi
  podczas wypełniania jego własnego formularza — nigdy nie jest wysyłane
  nigdzie poza tym żądaniem) z domyślną opcją „Wolę nie podawać”;
  wysłanie dołącza wybrane `child_id`.
- Lista recenzji: renderuj `child_age_years`, gdy obecne, jako małą
  przetłumaczoną linię pod recenzją, np. „Rodzic 4-latka” — nigdy nie
  renderuj imienia dziecka ani żadnego innego szczegółu identyfikującego,
  ponieważ API nigdy go w tym celu nie zwraca.
- i18n: `ratings.reviewingFor.label`, `.notSaid`,
  `ratings.childContext` (`"Parent of a {{age}}-year-old"`) — wszystkie
  cztery pliki lokalizacji.

#### Poza zakresem

- Dołączanie więcej niż jednego dziecka do pojedynczej oceny — jedno
  opcjonalne dziecko na ocenę, zgodnie z ustalonym już wzorcem
  wyboru pojedynczego dziecka „Dla Mii” z zadania 11.
- Jakiekolwiek wykorzystanie tego powiązania do rekomendacji/
  personalizacji poza wyświetlaniem — to rozszerzałoby zakres zadania
  11, a nie tego zadania.

#### Kryteria akceptacji

- Ocena wysłana z `child_id` pokazuje bieżący wiek dziecka (nie imię,
  nie id) na publicznej liście recenzji; ocena wysłana bez niego nie
  pokazuje linii z kontekstem dziecka, zgodnie z obecnym zachowaniem
  zadania 09.
- Użytkownik nie może dołączyć cudzego dziecka do swojej oceny (404,
  brak wycieku danych).
- Nic w publicznej odpowiedzi API dla ocen nigdy nie zawiera imienia ani
  id dziecka — wyłącznie wyliczony wiek.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
