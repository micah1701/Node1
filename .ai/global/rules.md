# Global Rules

Hard constraints and conventions for working in this repo. Read this before
making any change.

## Layering

- Prefer not to edit `src/core/` — it's the reusable "node1" framework
  layer. If you must change it, preserve backward compatibility.
- New product functionality belongs in `src/app/` (or a sibling top-level
  dir like `src/notifications/`, `src/lib/` when it doesn't fit the
  controller/route shape).
- `src/app/controllers/my-app.controller.ts` + `my-app.routes.ts` is a
  disposable framework scaffold, not a real feature — safe to delete or
  repurpose, and useful as the template for a new tool group (see
  [../skills/add-endpoint.md](../skills/add-endpoint.md)).

## Response & error conventions

- Every controller returns the `ApiResponse<T>` envelope
  (`{ success, data?, error?, message?, requestId? }`) and the `HttpStatus`
  enum, both from `src/core/types/index.ts` — never a raw ad-hoc JSON shape.
- Errors: `throw new ApiError(HttpStatus.X, 'message')` then `next(error)`.
  Never hand-format an error response — the centralized `errorHandler`
  (`src/core/middlewares/error.middleware.ts`) owns that and is mounted
  last in `src/index.ts`.
- Validate request bodies/params/query with `express-validator` chains
  passed through the `validate()` middleware
  (`src/core/middlewares/validation.middleware.ts`) — don't hand-roll
  validation in controllers.

## Auth

- Gate routes with `authenticate` from
  `src/core/middlewares/auth.middleware.ts` to require a valid JWT; it
  attaches `req.user: { id, email, role }`.
- `authorize(roles: string[])` must run **after** `authenticate` — it reads
  `req.user`.
- Public/unauthenticated endpoints (e.g. `POST /api/auth/register`,
  `POST /api/auth/login`, `GET /api/health`) are the exception, not the
  default — most endpoints should be behind `authenticate`.

## Config & secrets

- Read config through the single `config` object in
  `src/core/config/index.ts`, never `process.env` directly in
  controllers/services. New env-driven settings get a new namespaced key on
  that object (mirror the existing shape of `config.database`, `config.jwt`,
  `config.encryption`).
- `JWT_SECRET` and `MASTER_ENCRYPTION_KEY` are required at process boot —
  `config/index.ts` throws immediately if either is missing.

## Logging & redaction

- Never bypass the globally-mounted `requestLogger` audit middleware
  (`src/core/middlewares/requestLogger.middleware.ts`, mounted at
  `config.endpointPrefix` in `src/index.ts`) — it's this project's audit
  trail, encrypting and storing every request/response.
- If you add a new sensitive field name to any request/response body, add
  it to `SENSITIVE_FIELDS` in `requestLogger.middleware.ts` so it gets
  redacted from logs (matched case-insensitively as a substring).

## Payload size

- JSON/urlencoded body parsing has no explicit `limit` set (`src/index.ts`)
  — falls back to Express's default (~100kb). Raise it with
  `express.json({ limit: '...' })` if an endpoint needs larger payloads.

## Style

- No comments explaining *what* code does — good names carry that. Only
  comment the *why* when it's non-obvious (a hidden constraint, a
  workaround, a subtle invariant).
- No backwards-compatibility shims for unused code — delete it outright.
