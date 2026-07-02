# Family Activity Finder

Parent Activity Discovery — help parents quickly find the best activities for their children based on age, location, weather, available time, and interests. See [docs/discovery_summary.md](docs/discovery_summary.md) for the full product vision and MVP scope.

Monorepo: **FastAPI** on **AWS Lambda** (HTTP API + **Mangum**), **SQLModel** + **Alembic** against PostgreSQL, and a **Vite + React** SPA on **S3** + **CloudFront**.

## Layout

- `backend/` — Serverless stack, FastAPI app, Alembic migrations
- `frontend/` — React SPA (`VITE_API_BASE_URL` injected at build time)
- `scripts/` — Deploy, migration, and GitHub Actions env helpers
- `docs/` — Product discovery notes and AWS OIDC setup guide

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

Do not copy template `.env.gha` files — provision fresh AWS resources for this project.

## License

Add a license file if you open-source this project.
