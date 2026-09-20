---
status: todo
---

# 15. Review photo attachments

## Summary

Let parents attach up to a few photos to a rating/review submitted in task
09. This is the remaining half of the "ocenianie z mozliwoscia dodowania
opisu i zdjec" (rating with the ability to add a description and photos)
idea from `docs/backlog/IDEAS.md` — the description half (the `comment`
field) already shipped as part of task 09, which explicitly listed photo
attachments as out of scope.

## Why

`IDEAS.md` names photos as part of the same idea as the written review, not
a separate one — real-world photos of an activity (a park, a play area, a
class in session) are strong trust signals for other parents deciding
whether to go, on top of the star score and text.

## Backend changes

- Add object storage for uploaded images (reuse whatever storage backend
  the template/infra already provisions for file uploads, e.g. an S3-
  compatible bucket referenced in `backend/src/config.py`/infra scripts; if
  none exists yet, provision the smallest viable bucket and wire its
  name/region through the existing settings pattern rather than introducing
  a new config mechanism).
- `backend/src/apps/ratings/models.py` — new table `rating_photos`:
  `id: UUID` pk, `rating_id: UUID` (`foreign_key="ratings.id"`, indexed),
  `storage_key: str`, `created_at: datetime`. A rating can have zero or
  more photos (one-to-many, not a column on `ratings`).
- `backend/src/apps/ratings/routes.py`:
  - `POST /api/activities/{activity_id}/ratings/photos` — auth required,
    `multipart/form-data` upload, limited to image content types
    (`image/jpeg`, `image/png`, `image/webp`) and a size cap (e.g. 5 MB);
    422 on anything else. Requires the caller to already have a rating on
    this activity (create the rating first via task 09's endpoint, then
    attach photos to it) — 404 if they don't. Cap the count per rating
    (e.g. 5) — 422 past the cap. Stores the file, inserts a
    `rating_photos` row, returns its id + a servable URL.
  - `DELETE /api/activities/{activity_id}/ratings/photos/{photo_id}` — auth
    required, scoped to the caller's own rating; idempotent.
  - Extend `GET /api/activities/{activity_id}/ratings`'s per-rating shape
    with a `photos: [{"id", "url"}, ...]` array.
  - Extend `DELETE .../ratings/me` (task 09) to also delete any attached
    photo rows and their stored objects, so deleting a rating doesn't leave
    orphaned files.
- Tests: upload succeeds and appears in the `GET` listing, rejects
  non-image content types and oversized files, enforces the per-rating
  photo cap, deleting a rating cascades its photos, a user can't delete
  another user's photo.

## Data changes

- `make make-migrations` — new `rating_photos` table with FK to `ratings`.
- `make migrate`.

## Frontend changes

- `ActivityDetailPage.tsx`'s rating submission widget (task 09): after
  saving a rating, show a photo upload control (file input, multiple
  select up to the backend cap) that calls the new upload endpoint per
  file; show upload progress/errors per file, not a single blocking
  spinner for the whole batch.
- Reviews list: render each review's attached photos as a small thumbnail
  strip; clicking a thumbnail opens it larger (a simple lightbox/modal is
  enough — no gallery library needed for a handful of images).
- Allow the review author to delete their own attached photos individually
  from the same widget.
- i18n: `ratings.photos.add`, `.uploading`, `.tooLarge`, `.wrongType`,
  `.limitReached`, `.remove` — all four locale files.

## Out of scope

- Image moderation (inappropriate content scanning) — flagged as a
  follow-up once real usage surfaces the need, same rationale task 09 gave
  for review moderation generally.
- Client-side image cropping/editing — raw upload only for MVP.
- Photos attached anywhere other than a rating (e.g. directly on an
  `Activity` by an admin) — task 12's admin form is unaffected by this
  task.

## Acceptance criteria

- A signed-in user can attach photos to their own rating and see them in
  the public reviews list immediately.
- Uploading a non-image file or a file over the size cap is rejected with a
  translated error, not a silent failure or a crash.
- Deleting a rating (task 09's own-rating delete) leaves no orphaned photo
  rows or stored files.
- `make check` passes; `cd frontend && npm run build` succeeds.

---

## Tłumaczenie (PL)

### 15. Załączniki zdjęciowe do recenzji

#### Podsumowanie

Pozwól rodzicom dołączyć kilka zdjęć do oceny/recenzji wysłanej w zadaniu
09. To pozostała połowa pomysłu „ocenianie z możliwością dodawania opisu
i zdjęć” z `docs/backlog/IDEAS.md` — połowa dotycząca opisu (pole
`comment`) została już dostarczona w ramach zadania 09, które wprost
wymieniało załączniki zdjęciowe jako poza zakresem.

#### Dlaczego

`IDEAS.md` opisuje zdjęcia jako część tego samego pomysłu co pisemna
recenzja, a nie osobny pomysł — prawdziwe zdjęcia aktywności (park, plac
zabaw, zajęcia w trakcie) to silny sygnał zaufania dla innych rodziców
decydujących, czy tam pójść, oprócz oceny gwiazdkowej i tekstu.

#### Zmiany w backendzie

- Dodaj magazyn obiektów na przesyłane obrazy (wykorzystaj dowolny
  backend magazynowania, jaki już zapewnia szablon/infrastruktura dla
  przesyłania plików, np. bucket kompatybilny z S3 wskazany w
  `backend/src/config.py`/skryptach infrastruktury; jeśli jeszcze nic
  takiego nie istnieje, uruchom najmniejszy sensowny bucket i podepnij
  jego nazwę/region przez istniejący wzorzec ustawień, zamiast
  wprowadzać nowy mechanizm konfiguracji).
- `backend/src/apps/ratings/models.py` — nowa tabela `rating_photos`:
  `id: UUID` pk, `rating_id: UUID` (`foreign_key="ratings.id"`,
  indeksowane), `storage_key: str`, `created_at: datetime`. Ocena może
  mieć zero lub więcej zdjęć (relacja jeden-do-wielu, a nie kolumna w
  `ratings`).
- `backend/src/apps/ratings/routes.py`:
  - `POST /api/activities/{activity_id}/ratings/photos` — wymaga
    autoryzacji, przesyłanie `multipart/form-data`, ograniczone do
    typów obrazów (`image/jpeg`, `image/png`, `image/webp`) i limitu
    rozmiaru (np. 5 MB); 422 dla wszystkiego innego. Wymaga, aby
    wywołujący miał już ocenę dla tej aktywności (najpierw utwórz ocenę
    przez endpoint z zadania 09, potem dołącz do niej zdjęcia) — 404,
    jeśli jej nie ma. Ogranicz liczbę zdjęć na ocenę (np. 5) — 422 po
    przekroczeniu limitu. Zapisuje plik, wstawia wiersz
    `rating_photos`, zwraca jego id + dostępny URL.
  - `DELETE /api/activities/{activity_id}/ratings/photos/{photo_id}` —
    wymaga autoryzacji, ograniczone do własnej oceny wywołującego;
    idempotentne.
  - Rozszerz kształt każdej oceny w `GET
    /api/activities/{activity_id}/ratings` o tablicę `photos: [{"id",
    "url"}, ...]`.
  - Rozszerz `DELETE .../ratings/me` (zadanie 09), aby usuwał także
    wszelkie dołączone wiersze zdjęć i zapisane obiekty, tak aby
    usunięcie oceny nie zostawiało osieroconych plików.
- Testy: przesłanie kończy się sukcesem i pojawia się na liście `GET`,
  odrzuca typy inne niż obrazy i zbyt duże pliki, egzekwuje limit zdjęć
  na ocenę, usunięcie oceny kaskadowo usuwa jej zdjęcia, użytkownik nie
  może usunąć cudzego zdjęcia.

#### Zmiany w danych

- `make make-migrations` — nowa tabela `rating_photos` z FK do `ratings`.
- `make migrate`.

#### Zmiany we frontendzie

- Widget wysyłania oceny w `ActivityDetailPage.tsx` (zadanie 09): po
  zapisaniu oceny pokaż kontrolkę przesyłania zdjęć (pole pliku, wybór
  wielokrotny do limitu backendu), która wywołuje nowy endpoint
  przesyłania dla każdego pliku; pokazuj postęp/błędy przesyłania dla
  każdego pliku osobno, a nie jeden blokujący spinner dla całej paczki.
- Lista recenzji: renderuj dołączone zdjęcia każdej recenzji jako mały
  pasek miniatur; kliknięcie miniatury otwiera ją w większym rozmiarze
  (prosty lightbox/modal wystarczy — bez potrzeby biblioteki galerii dla
  garstki zdjęć).
- Pozwól autorowi recenzji usuwać własne dołączone zdjęcia pojedynczo z
  tego samego widgetu.
- i18n: `ratings.photos.add`, `.uploading`, `.tooLarge`, `.wrongType`,
  `.limitReached`, `.remove` — wszystkie cztery pliki lokalizacji.

#### Poza zakresem

- Moderacja obrazów (skanowanie nieodpowiednich treści) — oznaczone jako
  kontynuacja, gdy realne użycie pokaże taką potrzebę, z tym samym
  uzasadnieniem, jakie zadanie 09 podało dla moderacji recenzji ogółem.
- Przycinanie/edycja obrazów po stronie klienta — dla MVP tylko surowe
  przesyłanie.
- Zdjęcia dołączane gdziekolwiek indziej niż do oceny (np. bezpośrednio
  do `Activity` przez administratora) — to zadanie nie wpływa na
  formularz administracyjny z zadania 12.

#### Kryteria akceptacji

- Zalogowany użytkownik może dołączyć zdjęcia do własnej oceny i od razu
  zobaczyć je na publicznej liście recenzji.
- Przesłanie pliku, który nie jest obrazem, lub pliku przekraczającego
  limit rozmiaru jest odrzucane z przetłumaczonym błędem, a nie cichym
  niepowodzeniem czy błędem aplikacji.
- Usunięcie oceny (usuwanie własnej oceny z zadania 09) nie pozostawia
  osieroconych wierszy zdjęć ani zapisanych plików.
- `make check` przechodzi; `cd frontend && npm run build` kończy się
  sukcesem.
