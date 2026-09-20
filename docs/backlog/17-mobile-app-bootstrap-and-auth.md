---
status: todo
---

# 17. Mobile app bootstrap and auth

## Summary

Scaffold a new React Native + Expo app in `mobile/` and port the sign-up /
sign-in / password-recovery / OAuth flows already working on the web
frontend, talking to the same backend API. This is the foundation task for
the "Build iOS and Android apps in React Native + Expo. Mobile apps and web
interface should use the same backend API" line item added to
`docs/backlog/IDEAS.md` — every later mobile task builds a screen on top of
this shell.

## Why

The backend's auth is already bearer-token based (`Authorization: Bearer
<token>`, decoded via `decode_token`/`HTTPBearer` in
`backend/src/apps/users/routes.py`), not cookie/session based — the exact
shape a native client needs, with no web-specific session workaround to
undo. That makes "same backend API" close to free; the actual work is a
new native client, not a new API.

## Backend changes

- `backend/src/apps/users/oauth.py`: the Google/Facebook OAuth redirect
  flow currently assumes a web redirect URI. Add the mobile app's custom
  URL scheme (`familyactivityfinder://auth-callback`, registered in the
  Expo `app.json`/`app.config.ts` from this task) to the provider's
  allowed redirect URIs / the backend's own allow-list, alongside the
  existing web one — do not replace it, both clients need to keep working.
- No other backend changes: `GET /api/auth/me` and the existing
  register/verify/login/password-recovery endpoints are consumed as-is.

## Data changes

None.

## Frontend changes

- New `mobile/` directory at the repo root (sibling to `backend/` and
  `frontend/`): `npx create-expo-app` (TypeScript template), Expo Router
  or React Navigation for screen navigation (pick one and use it
  consistently in every later mobile task).
- Shared config: point the app at the same backend base URL pattern the
  web frontend uses (env-driven, e.g. `EXPO_PUBLIC_API_BASE_URL`), so
  local dev, staging, and prod builds can target the right backend without
  code changes.
- Token storage: use `expo-secure-store` (not `AsyncStorage`, which is
  unencrypted) to persist the bearer token across app restarts — the
  native equivalent of the web app's `localStorage` token handling.
- Port the web app's auth screens/logic
  (`frontend/src/auth/`) to native screens: sign up (email + verification
  code), sign in, password recovery (request + verify code + reset), sign
  out, and "continue with Google/Facebook" using `expo-auth-session` (its
  redirect must match the custom scheme registered on the backend above).
- i18n: reuse the same translation keys/values as the web app's
  `frontend/src/i18n/locales/*.json` (either via a shared package the two
  apps both import, or by copying the JSON files into `mobile/` and
  keeping them in sync manually for now — pick the shared-package route if
  the added tooling cost is small, otherwise note the manual-sync tradeoff
  explicitly in the PR).
- A minimal authenticated "home" placeholder screen (just enough to prove
  `GET /api/auth/me` works end-to-end) — the real home/browse screen is
  task 18.

## Out of scope

- Any feature screen beyond auth (browse, favorites, ratings, children,
  admin) — tasks 18–20 build those.
- Admin tooling on mobile — task 12's admin screens stay web-only; no
  mobile task ports them (an admin managing content is expected to use a
  desktop browser).
- App Store / Play Store submission, signing, and CI build pipelines —
  follow-up once there's a feature-complete app worth submitting.
- Push notifications — not requested by `IDEAS.md`; would be a separate
  idea/task if it comes up later.

## Acceptance criteria

- A fresh install of the app can sign up a new account (email +
  verification code), sign in, and reach the placeholder home screen.
- Force-quitting and reopening the app keeps the user signed in (token
  persisted via `expo-secure-store`).
- Signing in with Google/Facebook completes the OAuth round trip and lands
  back in the app authenticated.
- Signing out clears the stored token and returns to the sign-in screen.
- The mobile app and the web frontend both authenticate against the same
  backend deployment with no backend fork required.

---

## Tłumaczenie (PL)

### 17. Uruchomienie aplikacji mobilnej i autoryzacja

#### Podsumowanie

Zbuduj nową aplikację React Native + Expo w katalogu `mobile/` i przenieś
przepływy rejestracji / logowania / odzyskiwania hasła / OAuth działające
już w aplikacji webowej, komunikujące się z tym samym backendowym API. To
zadanie fundamentalne dla pozycji „Zbuduj aplikacje iOS i Android w React
Native + Expo. Aplikacje mobilne i interfejs webowy powinny korzystać z
tego samego backendowego API”, dodanej do `docs/backlog/IDEAS.md` — każde
kolejne zadanie mobilne buduje ekran na tym fundamencie.

#### Dlaczego

Autoryzacja backendu jest już oparta na tokenach bearer
(`Authorization: Bearer <token>`, dekodowana przez
`decode_token`/`HTTPBearer` w `backend/src/apps/users/routes.py`), a nie
na ciasteczkach/sesjach — to dokładnie taki kształt, jakiego potrzebuje
klient natywny, bez konieczności obchodzenia rozwiązania specyficznego
dla weba. Dzięki temu „to samo backendowe API” jest niemal darmowe;
prawdziwą pracą jest nowy klient natywny, a nie nowe API.

#### Zmiany w backendzie

- `backend/src/apps/users/oauth.py`: obecny przepływ OAuth
  Google/Facebook zakłada webowy URI przekierowania. Dodaj niestandardowy
  schemat URL aplikacji mobilnej
  (`familyactivityfinder://auth-callback`, rejestrowany w
  `app.json`/`app.config.ts` Expo w ramach tego zadania) do listy
  dozwolonych URI przekierowania dostawcy / listy dozwolonych po stronie
  backendu, obok istniejącego webowego — nie zastępuj go, oba klienty
  muszą nadal działać.
- Brak innych zmian w backendzie: `GET /api/auth/me` i istniejące
  endpointy rejestracji/weryfikacji/logowania/odzyskiwania hasła są
  wykorzystywane bez zmian.

#### Zmiany w danych

Brak.

#### Zmiany we frontendzie

- Nowy katalog `mobile/` w katalogu głównym repozytorium (obok
  `backend/` i `frontend/`): `npx create-expo-app` (szablon TypeScript),
  Expo Router lub React Navigation do nawigacji między ekranami (wybierz
  jedno i używaj konsekwentnie w każdym kolejnym zadaniu mobilnym).
- Współdzielona konfiguracja: skieruj aplikację na ten sam wzorzec
  bazowego URL backendu, jakiego używa aplikacja webowa (sterowany
  zmiennymi środowiskowymi, np. `EXPO_PUBLIC_API_BASE_URL`), tak aby
  lokalny development, staging i buildy produkcyjne mogły celować we
  właściwy backend bez zmian w kodzie.
- Przechowywanie tokena: użyj `expo-secure-store` (nie `AsyncStorage`,
  który jest nieszyfrowany), aby zachować token bearer między
  restartami aplikacji — natywny odpowiednik obsługi tokena przez
  `localStorage` w aplikacji webowej.
- Przenieś ekrany/logikę autoryzacji z aplikacji webowej
  (`frontend/src/auth/`) na ekrany natywne: rejestracja (e-mail + kod
  weryfikacyjny), logowanie, odzyskiwanie hasła (żądanie + weryfikacja
  kodu + reset), wylogowanie oraz „kontynuuj przez Google/Facebook” za
  pomocą `expo-auth-session` (jego przekierowanie musi pasować do
  niestandardowego schematu zarejestrowanego w backendzie powyżej).
- i18n: wykorzystaj te same klucze/wartości tłumaczeń co pliki aplikacji
  webowej `frontend/src/i18n/locales/*.json` (albo przez współdzielony
  pakiet, który importują obie aplikacje, albo przez skopiowanie plików
  JSON do `mobile/` i ręczne utrzymywanie ich w spójności — wybierz
  wariant współdzielonego pakietu, jeśli dodatkowy koszt narzędziowy
  jest niewielki, w przeciwnym razie jasno opisz w PR kompromis
  ręcznej synchronizacji).
- Minimalny zalogowany ekran „główny” zastępczy (tylko na tyle, by
  udowodnić, że `GET /api/auth/me` działa od początku do końca) —
  prawdziwy ekran główny/przeglądania to zadanie 18.

#### Poza zakresem

- Jakikolwiek ekran funkcjonalny poza autoryzacją (przeglądanie,
  ulubione, oceny, dzieci, administracja) — budują je zadania 18–20.
- Narzędzia administracyjne na urządzeniach mobilnych — ekrany
  administracyjne z zadania 12 pozostają wyłącznie webowe; żadne
  zadanie mobilne ich nie przenosi (oczekuje się, że administrator
  zarządzający treścią użyje przeglądarki na komputerze).
- Wysyłka do App Store / Play Store, podpisywanie i pipeline’y CI do
  budowania — kontynuacja po tym, jak powstanie aplikacja z pełną
  funkcjonalnością, wartą wysłania.
- Powiadomienia push — nie wymagane przez `IDEAS.md`; byłyby osobnym
  pomysłem/zadaniem, gdyby pojawiły się później.

#### Kryteria akceptacji

- Świeża instalacja aplikacji pozwala zarejestrować nowe konto (e-mail +
  kod weryfikacyjny), zalogować się i dotrzeć do zastępczego ekranu
  głównego.
- Wymuszone zamknięcie i ponowne otwarcie aplikacji utrzymuje
  zalogowanie (token przechowywany przez `expo-secure-store`).
- Logowanie przez Google/Facebook kończy pełny przepływ OAuth i wraca do
  aplikacji w stanie zalogowanym.
- Wylogowanie czyści zapisany token i wraca do ekranu logowania.
- Aplikacja mobilna i aplikacja webowa autoryzują się względem tego
  samego wdrożenia backendu, bez potrzeby jego rozgałęzienia.
