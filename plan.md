# Implementation Plan

Phased build for the [assignment](./docs/assignment.md). Stack: **NestJS REST API** + **Angular CLI** + MySQL + Redis.

| Phase | Status | Deliverable |
|-------|--------|-------------|
| 0 | Done | Nx monorepo, NestJS + Angular CLI, MySQL + Redis, `GET /api/health` |
| 1 | Done | TypeORM entities, domain module stubs, sample CSVs, inventory seed |
| 2 | Done | Coordinator happy path → `PLACED` |
| 3 | Done | Compensation → `CANCELLED` / `NEEDS_ATTENTION` |
| 4 | Done | Idempotency cache, recovery sweeper |
| 5 | Done | Streaming CSV ingestion with dedup |
| 6 | Done | `markShipped` + 15-min notification cron |
| 7 | Done | **REST API** + Angular Material UI |
| 8 | Done | Unit tests, RUN.md checker guide |

## Key commands

```bash
npm install
npm run generate:data    # regenerate data/*.csv
npm run validate:data    # verify CSV format before ingest
npm run start:api        # NestJS REST API → :3000
npm run start:web        # Angular CLI → :4200
npm start                # both in parallel
npm run ingest           # load ~2500 orders
npm test
```

## REST endpoints (UI contract)

```
GET  /api/health
GET  /api/orders
GET  /api/orders/:id
GET  /api/orders/:id/steps
POST /api/orders/:id/retry-compensation
POST /api/orders/:id/ship
```

## Documentation

- [docs/PRD.md](./docs/PRD.md) — requirements and grading checklist
- [docs/backend-architecture.md](./docs/backend-architecture.md) — saga engine, data model
- [docs/design-frontend.md](./docs/design-frontend.md) — Angular UI spec
- [RUN.md](./RUN.md) — local setup and troubleshooting
