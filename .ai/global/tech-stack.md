# Tech Stack

## Language & framework

- Node.js + TypeScript 5.8 (`tsconfig.json`: `target: es2020`,
  `module: commonjs`, `rootDir: src`, `outDir: dist`, strict mode).
- Express 4.21.
- Package name in `package.json` is `typescript-api-framework`; this
  deployment is built on that internal "node1" template.

## Key dependencies

- **HTTP/security**: `helmet`, `cors`, `morgan`, `express-validator`
- **Auth**: `jsonwebtoken`, `bcryptjs`
- **DB**: `mysql2`, `@supabase/supabase-js`
- **Crypto**: `tweetnacl` + `tweetnacl-util` (X25519), Node's built-in `crypto` (RSA/AES/Ed25519)
- **Misc**: `winston` (logging), `uuid`, `dotenv`

## npm scripts (`package.json`)

| Script | Command | Purpose |
|---|---|---|
| `dev` | `ts-node src/index.ts` | Run in dev without a build step |
| `build` | `tsc` | Compile `src/` → `dist/` |
| `start` | `node dist/index.js` | Run the compiled build |
| `lint` | `eslint src/**/*.ts` | Lint (no eslint config file currently checked in at repo root — verify before relying on it) |
| `test` | `jest` | Run tests (see Testing below) |
| `db:setup` | `ts-node src/core/scripts/setupDatabase.ts` | Bootstrap MySQL schema |

## Deployment

Just a plain Node process — no process manager, containerization, or
deploy scripts are wired up in this template. `npm run build && npm start`
is the base "compile and run" flow; add PM2/systemd/Docker/etc. as your
actual deployment target requires.

## Database

Dual-backend, chosen by the `DATABASE_TYPE` env var (default: `postgres` if
unset — see `src/core/config/index.ts`):
- `mysql` → `MySQLDatabase` (raw SQL via `mysql2` pool, real `execute()`)
- `postgres` (default) → `SupabaseDatabase` (`@supabase/supabase-js`
  client; `execute()` throws — requires adding typed per-table methods)

Both implement `DatabaseInterface` in `src/core/utils/db.ts`. See
[../core/patterns.md](../core/patterns.md) for how to use it.

## Testing

Jest + ts-jest is fully configured (`jest.config.ts`; `testMatch`:
`**/__tests__/**/*.ts`, `*.test.ts`, `*.spec.ts`) but **zero test files
currently exist anywhere in `src/`** — there's no established testing
convention to follow yet. If you add the first test file, establish the
pattern deliberately rather than guessing at one.

## Env

`.env` (gitignored) / `.env.example` (template) at repo root, loaded once
via `dotenv` in `src/core/config/index.ts`.
