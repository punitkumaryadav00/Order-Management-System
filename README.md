# Order Processing System (OMS)

Nx monorepo implementing the [assignment brief](./docs/assignment.md) with a **NestJS REST API** backend and an **Angular CLI** frontend.

## Stack (locked)

| Layer | Technology |
|-------|------------|
| Backend | **NestJS** modular monolith — `apps/api` |
| API | **REST** JSON over HTTP — `/api/*` |
| Frontend | **Angular CLI** — `apps/web` (Material UI) |
| Database | **MySQL 8** + TypeORM |
| Async / cache | **Redis** + BullMQ (internal orchestration only) |
| Monorepo | Nx 23 — shared DTOs in `libs/shared/*` |

The four assignment “services” (Order, Inventory, Payment, Shipping) are **NestJS modules** in one app, not separate deployables. The Angular UI talks to the backend **only via REST**.

## Quick start

```bash
npm install
cp .env.example .env          # set MySQL + Redis
npm run generate:data         # create sample CSVs
npm run validate:data         # verify CSV format
npm start                     # NestJS :3000 + Angular :4200
```

In another terminal:

```bash
npm run ingest                # load ~2500 orders
```

- API health: http://localhost:3000/api/health  
- UI: http://localhost:4200  

See [RUN.md](./RUN.md) for full setup, troubleshooting, and grading checklist.

## REST API

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/health` | Infra check |
| `GET` | `/api/orders` | List orders (paginated, filterable) |
| `GET` | `/api/orders/:id` | Order detail |
| `GET` | `/api/orders/:id/steps` | Step timeline |
| `POST` | `/api/orders/:id/retry-compensation` | Retry failed undo |
| `POST` | `/api/orders/:id/ship` | Mark shipped |

## Documentation

| Doc | Description |
|-----|-------------|
| [docs/assignment.md](./docs/assignment.md) | Original assignment brief |
| [docs/PRD.md](./docs/PRD.md) | Expanded requirements |
| [docs/backend-architecture.md](./docs/backend-architecture.md) | NestJS modules, saga, data model |
| [docs/design-frontend.md](./docs/design-frontend.md) | Angular UI spec |
| [plan.md](./plan.md) | Build phases |

## Projects

| Project | Command | Description |
|---------|---------|-------------|
| `api` | `npm run start:api` | NestJS REST API |
| `web` | `npm run start:web` | Angular CLI dev server |
| `enums`, `types` | — | Shared libraries |

```bash
npx nx graph    # dependency graph
```
