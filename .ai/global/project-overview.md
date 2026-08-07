# Project Overview

## What this is

**node1** is a base template for building Node.js/TypeScript/Express REST
APIs: a reusable core framework (JWT auth, DB abstraction, encryption,
key-value storage, SSH key generation, request/audit logging) plus a thin
app layer meant to be replaced with your actual product.

## Two-layer architecture

```
src/
├── core/                   # "node1" — reusable framework layer
│   ├── config/index.ts     # single config object, built from process.env
│   ├── controllers/        # auth, keyValue, sshKeys, apiLogs
│   ├── middlewares/        # auth, error, requestLogger, validation
│   ├── routes/             # auth, keyValue, sshKeys, apiLogs, core.routes (aggregator)
│   ├── scripts/setupDatabase.ts
│   ├── types/index.ts      # ApiResponse<T>, HttpStatus, User, etc.
│   └── utils/               db.ts, encryption.ts, jwt.utils.ts, logger.ts
│
├── app/                    # where your product's features go
│   ├── README.md            # endpoint reference — overwrite for your app
│   ├── controllers/         my-app (disposable scaffold)
│   └── routes/               my-app, index (aggregator)
│
├── routes/index.ts         # mounts coreRoutes then appRoutes, then a 404 catch-all
└── index.ts                # entry point: middleware stack, static files, routes, error handler
```

## Route mounting

`src/routes/index.ts`: `router.use('/', coreRoutes)` then
`router.use('/', appRoutes)`, both under `config.endpointPrefix` (default
`/api`, set via `ENDPOINT_PREFIX`).

| Prefix | Source |
|---|---|
| `/api/health`, `/api/auth`, `/api/key-values`, `/api/ssh-keys`, `/api/api-logs` | `src/core/routes/core.routes.ts` |
| `/api/my-app` | `src/app/routes/index.ts` (each sub-router owns its own prefix) |

`GET /api/health` is also defined directly in `src/index.ts` (before
`routes` is mounted) — it works even if the database is down, reporting
`database: 'connected' | 'disconnected'`.

## Request lifecycle

```
helmet (CSP) → cors → express.json/urlencoded → morgan (dev log)
  → requestLogger (assigns requestUuid, encrypts+audits, mounted at config.endpointPrefix)
  → [per-route] authenticate (JWT) → authorize(roles) if applicable
  → [per-route] validate(expressValidatorChains)
  → controller → ApiResponse<T> JSON response
  → errorHandler (on thrown ApiError / Error) — mounted last in src/index.ts
```
