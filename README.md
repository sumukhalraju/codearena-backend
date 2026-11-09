# CodeArena backend

This repository is the API, the match authority, and the judge worker. It is nested in the parent repository `codearena` as the `backend` submodule.

The parent repository decides what is true. This repository makes PostgreSQL, Redis, and a single-use container agree with that. Do not “simplify” a rule in a handler. Change `codearena` first.

Start with the parent files:

- `docs/00-start-here.md`
- `docs/02-decisions.md`
- `docs/03-architecture.md`
- `docs/04-match-engine.md`
- `docs/05-judge-and-sandbox.md`
- `docs/10-backend.md` (the build manual)
- `docs/06-http-api.md` and `contracts/openapi.yaml`
- `docs/07-realtime.md`
- `contracts/schema.sql`
- `contracts/constants.json`

## Two processes

| Process | Listens | Does |
| --- | --- | --- |
| `api` | `:4000` | HTTP and WebSocket. No Docker socket. |
| `worker` | `:4001` on the internal network only | Matcher tick, timers, judge, reconciler |

Migrations are a one-shot command, not something the API runs on boot.

## Stack to scaffold

- Node.js 22, pnpm 9, TypeScript strict
- NestJS
- Prisma and PostgreSQL 16
- Redis 7 and BullMQ
- argon2id
- Zod at the boundary
- dockerode in the worker only
- Vitest, Supertest, Testcontainers

`packages/domain` is pure TypeScript. The worked scoring example in the parent match-engine doc is a unit test here. If the test and the doc disagree, the constants file and the doc win, and the code changes.

## Layout

The directory tree to create is in the parent `docs/10-backend.md`. Follow it so reviews have a stable map: `apps/api`, `apps/worker`, `packages/domain`, `packages/db`, `sandbox/`.

## Commands, once `package.json` exists

```powershell
pnpm install
pnpm db:migrate
pnpm dev:api
pnpm dev:worker
```

Local PostgreSQL and Redis come from the parent file `deploy/docker-compose.data.yml`. Copy the parent `deploy/env.example` to `.env` in this repository.

```powershell
pnpm test
pnpm typecheck
```

## Hard rules

- `users.rating` is assigned in one function, inside the finalize transaction, and nowhere else.
- Player responses go through one projection function. Do not return a Prisma row from a controller.
- The BullMQ payload for a judge job is `{ submissionId }` only. Hidden tests are loaded from PostgreSQL.
- Every container uses the flag list in `docs/05-judge-and-sandbox.md`. One submission phase, one new container, then delete it.
- `internal_error` does not consume an attempt.
- Sample runs do not call the scoring function.
- Vendor `contracts/` from the parent when the app is scaffolded. `scripts/check-contracts.ps1` in the parent fails on drift.

## What is not in this repo yet

There is no application code, no Dockerfile, and no language image. Milestone 0 in the parent `docs/14-delivery.md` is the health-check skeleton. The Python image should be proven in milestone 2, before code rounds are player-facing.
