# How to Run — Order Processing System

NestJS REST API (`apps/api`) + Angular CLI UI (`apps/web`). See [docs/assignment.md](./docs/assignment.md) for requirements.

## Prerequisites

- Node.js 22+ (24 recommended; Angular 22 requires `^24.15.0` for full engine compatibility)
- npm 10+
- **MySQL 8+** — installed and running locally
- **Redis** — installed and running locally (required for BullMQ)

**No Docker.** Install MySQL and Redis directly on your machine (or use a remote/hosted instance). The app connects via environment variables.

## Local services (no Docker)

### MySQL

```bash
# Ubuntu/Debian example
sudo apt update && sudo apt install mysql-server
sudo systemctl start mysql

# Create database and user
mysql -u root -p
```

```sql
CREATE DATABASE oms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'oms'@'localhost' IDENTIFIED BY 'oms';
GRANT ALL PRIVILEGES ON oms.* TO 'oms'@'localhost';
FLUSH PRIVILEGES;
```

### Redis

```bash
# Ubuntu/Debian example
sudo apt install redis-server
sudo systemctl start redis-server
redis-cli ping   # should return PONG
```

### Environment variables

Copy the example file and adjust if needed:

```bash
cp .env.example .env
```

Default values assume local MySQL and Redis on standard ports.

You can connect using either split variables (`DATABASE_HOST`, etc.) or `DATABASE_URL` / `REDIS_URL`. If your password contains special characters (e.g. `@`), either use split variables or URL-encode the password in `DATABASE_URL`.

## Sample data

Generate and validate CSV files before ingestion:

```bash
npm run generate:data   # writes data/sample_inventory.csv + data/orders_bulk.csv
npm run validate:data   # parses full CSV (must pass before ingest)
```


```bash
npm install
```

## Database schema

The API uses TypeORM with `synchronize: true` in development (tables are created automatically on first boot).

For production-style setup, run the initial migration instead:

```bash
npx typeorm migration:run -d apps/api/src/database/data-source.ts
```

Migration file: `apps/api/src/database/migrations/1730000000000-InitialSchema.ts`

## Development

Start both apps (recommended):

```bash
npm start
# or
npm run dev:all
```

Or individually:

```bash
npm run start:api   # NestJS REST API → http://localhost:3000/api
npm run start:web   # Angular CLI dev server → http://localhost:4200
```

The Angular dev server proxies `/api` to the NestJS backend at `http://localhost:3000`.

On first API boot, inventory is seeded from `data/sample_inventory.csv` (20 products).

## Bulk order ingestion

Build the API, then ingest the sample CSV (~2,500 orders):

```bash
npm run ingest
# or
npx nx run api:ingest
```

This streams `data/orders_bulk.csv`, deduplicates by `external_order_id`, and enqueues new sagas for processing.

## Build (esbuild)

Both apps use **esbuild** bundlers:

| App | Bundler | Command | Output |
|-----|---------|---------|--------|
| `api` | `@nx/esbuild:esbuild` + `esbuild-plugin-tsc` (NestJS decorators) | `npm run build:api` | `dist/apps/api/main.cjs` |
| `web` | `@angular/build:application` (esbuild) | `npm run build:web` | `dist/apps/web/browser/` |

Build everything:

```bash
npm run build
```

## Tests

```bash
npm test
# or
npm run test:api
# or
npx nx test api
```

Tests cover:

- Domain idempotency (execute twice = no-op)
- Compensation scoping (only SUCCEEDED steps compensated)
- Saga state transitions (PLACED, CANCELLED, NEEDS_ATTENTION)
- CSV ingestion deduplication

## Nx Commands

```bash
npx nx serve api
npx nx serve web
npx nx build api
npx nx build web
npx nx run api:ingest
npx nx run-many -t build
npx nx graph
```

## Workspace Layout

```
apps/api/          NestJS modular monolith (esbuild)
apps/web/          Angular SPA (esbuild)
libs/shared/enums/ @oms/shared-enums
libs/shared/types/ @oms/shared-types
data/              CSV input files (orders_bulk.csv, sample_inventory.csv)
docs/              PRD and architecture documentation
```

## Verify API

After `npm run start:api`:

```bash
curl http://localhost:3000/api/health
# {"mysql":"ok","redis":"ok","status":"ok"}
```

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `mysql: "error"` in health | MySQL not running or wrong credentials | Start MySQL; verify `.env` matches your user/password; create `oms` database |
| `redis: "error"` in health | Redis not running | `sudo systemctl start redis-server`; `redis-cli ping` → PONG |
| Ingest crashes immediately | Malformed CSV | Run `npm run validate:data`; regenerate with `npm run generate:data` |
| Ingest inserts 0 rows | All orders already exist | Expected on re-ingest; truncate `order_saga` to reload |
| UI shows no orders | Ingest not run or API down | `curl /api/health`; then `npm run ingest` |
| Orders stuck `IN_PROGRESS` | Redis or workers not processing | Ensure API is running (workers live in the API process) |

**Correct startup order:** MySQL + Redis → `npm run start:api` (wait for healthy) → `npm run ingest` → open UI.

## Verify UI

Open http://localhost:4200 — orders list with filters, detail view with step timeline, retry compensation, and mark shipped.

## Checker / grader quick start

1. Install Node.js, MySQL 8, and Redis locally (no Docker required).
2. Create the `oms` database (see above).
3. `cp .env.example .env` and set credentials.
4. `npm install`
5. `npm run generate:data && npm run validate:data`
6. `npm run build`
7. `npm test` — all unit tests should pass.
8. `npm start` — open http://localhost:4200
9. `curl http://localhost:3000/api/health` — verify MySQL + Redis connectivity.
10. `npm run ingest` — load bulk orders and watch saga processing in the UI.
11. `npm run ingest` again — expect 0 inserted, ~2500 skipped (dedup test).
12. Filter by `IN_PROGRESS`, `PLACED`, `CANCELLED`, `NEEDS_ATTENTION` to verify saga flows.
13. On a `PLACED` order, use **Mark Shipped**; notification cron runs every 15 minutes.

**SQL sanity checks:**

```sql
SELECT status, COUNT(*) FROM order_saga GROUP BY status;
SELECT COUNT(*) FROM order_step_log WHERE phase = 'EXECUTE';
```

## Documentation

See [docs/README.md](./docs/README.md) for PRD, backend architecture, and frontend design.
