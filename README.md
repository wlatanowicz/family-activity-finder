# Family Activity Finder

Parent Activity Discovery — help parents quickly find the best activities for their children based on age, location, weather, available time, and interests. See [docs/discovery_summary.md](docs/discovery_summary.md) for the full product vision and MVP scope.

Monorepo: **FastAPI** on **AWS Lambda** (HTTP API + **Mangum**), **SQLModel** + **Alembic** against PostgreSQL, and a **Vite + React** SPA on **S3** + **CloudFront**.

## Layout

- `backend/` — Serverless stack, FastAPI app, Alembic migrations
- `frontend/` — React SPA (`VITE_API_BASE_URL` injected at build time)
- `scripts/` — Deploy, migration, Cloudflare DNS, ACM provisioning, and GitHub Actions env helpers
- `docs/` — Product discovery notes, AWS OIDC setup, [micro-saas-template origin](docs/micro-saas-template-origin.md) (tracked commit for future template updates), and [downstream template tracking](docs/downstream-template-tracking.md)

## Prerequisites

- [uv](https://docs.astral.sh/uv/) (Python toolchain and lockfile-driven installs)
- Node.js 22+ and npm (Serverless CLI + frontend build)
- Python 3.11 (matches `serverless.yml` runtime)
- Docker (for local full-stack dev and backend integration tests)
- PostgreSQL URL for `DATABASE_URL` when using DB-backed routes

## Quick start (local)

**Full stack with Docker Compose:**

```bash
make docker-start
```

This starts Postgres, the backend API on port 8000, and the frontend on port 5173.

**Backend tests and checks:**

```bash
make test-be    # ephemeral Postgres + pytest
make check      # tests + lint + lambda requirements export
```

**Frontend build:**

```bash
cd frontend && npm install && npm run build
```

## Local environment files

Copy the example env files before running outside Docker Compose:

```bash
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
```

Set `JWT_SECRET` in `backend/.env` (required when `DATABASE_URL` is set). Set `VITE_API_BASE_URL=http://localhost:8000` in `frontend/.env` for local dev.

## Database migrations

**Local:**

```bash
make migrate
# or: cd backend && uv run alembic upgrade head
```

**After model changes:**

```bash
make make-migrations
```

## What ships today

The scaffold includes auth (password + OAuth), email notifications, a scheduler, and a demo `items` CRUD app to verify the stack end-to-end. Domain-specific activity discovery features will replace the demo app in a later milestone.

## Deployment (later milestone)

AWS deployment, GitHub Actions secrets, and DNS setup are documented in [docs/github-actions-aws-oidc.md](docs/github-actions-aws-oidc.md). Bootstrap GitHub secrets from AWS with:

```bash
AWS_PROFILE=your-profile ./scripts/collect-gha-env.sh
./scripts/push-gha-env.sh
```

Provision ACM certificates (frontend in us-east-1, API in deploy region) with Cloudflare DNS validation:

```bash
AWS_PROFILE=your-profile ./scripts/provision-acm-certs.sh
```

Do not copy template `.env.gha` files — provision fresh AWS resources for this project.

## GitHub Actions

### `Mobile Android test` (`.github/workflows/mobile-android-test.yml`)

Builds an installable Android test APK on pushes to `main` that touch `mobile/`, `packages/`, the root package manifests, or this workflow, and on `workflow_dispatch`. The APK is uploaded as the **`android-test-apk`** artifact (90 days). It is a release build signed with the Expo debug keystore (`android` / `androiddebugkey`), so it can be sideloaded. That signature is not a Play App Signing key.

`EXPO_PUBLIC_API_BASE_URL` is baked in at build time, in this order: the manual **`api_base_url`** input, the **`EXPO_PUBLIC_API_BASE_URL`** repository variable, `https://` plus **`API_DOMAIN_NAME`**, then `https://api.<FRONTEND_DOMAIN_NAME>`. The job fails when none of those is set. `android.versionCode` comes from the GitHub run number so a newer APK can replace the previous install.

### `Mobile EAS build` (`.github/workflows/mobile-eas-build.yml`)

Disabled (`if: false`). The job would run `eas build --platform android --profile preview` for an internal APK. It does not run until `if: false` is removed. To turn it on:

1. Remove `if: false` from the job.
2. Add the **`EXPO_TOKEN`** secret.
3. Run `eas init` in `mobile/` and commit that app’s `extra.eas.projectId`. Project ids are per Expo project, so do not copy one from another repo.
4. Store **`EXPO_PUBLIC_API_BASE_URL`** as an EAS environment variable for the preview profile. GitHub Actions environment variables are not forwarded to the EAS build worker.
5. Generate Android credentials once with `eas credentials`. The command stays `--platform android`.

### `Mobile iOS device` (`.github/workflows/mobile-ios-device.yml`)

Disabled (`if: false`). The job would archive a signed ad hoc IPA for a physical iPhone and upload it as **`ios-device-ipa`**. It is not a TestFlight or App Store upload. The IPA installs only on devices whose UDIDs are in the provisioning profile, and that profile’s App ID must match `ios.bundleIdentifier` (`com.latanowicz.familyactivityfinder.app`). To turn it on:

1. Remove `if: false` from the job.
2. Enroll in the Apple Developer Program and register the test device.
3. Add these secrets: **`IOS_DISTRIBUTION_CERTIFICATE_BASE64`** (Apple distribution `.p12`, base64-encoded), **`IOS_DISTRIBUTION_CERTIFICATE_PASSWORD`**, **`IOS_PROVISIONING_PROFILE_BASE64`** (ad hoc `.mobileprovision`, base64-encoded), and **`IOS_TEAM_ID`**.

Do not commit the certificate or the provisioning profile.

## License

Add a license file if you open-source this project.
