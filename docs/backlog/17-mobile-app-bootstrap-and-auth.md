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
