# Core Patterns

Structural conventions from `src/core/` that every controller in this repo
should follow.

## Database (`src/core/utils/db.ts`)

```ts
interface DatabaseInterface {
  execute(query: string, params?: any[]): Promise<any>;
  getTableName(tableName: string): string;
}
```

Two implementations, selected once at module load by `config.database.type`
(`DATABASE_TYPE` env var, default `postgres`):

- **`MySQLDatabase`** — real `execute(sql, params)` via a `mysql2` pool.
- **`SupabaseDatabase`** — `execute()` **throws**
  (`'Direct SQL execution not supported with Supabase. Use specific
  methods.'`). Add typed methods per table instead (see
  `insertUser`/`findUserByEmail`/`insertKeyValue`/etc. in `db.ts` for the
  pattern).

Table names are prefixed via `config.database.tablePrefix`
(`TABLE_PREFIX` env var):

```ts
import { db } from '../../core/utils/db';
const tableName = db.getTableName('my_table'); // `${TABLE_PREFIX}my_table`
const result = await db.execute(`SELECT * FROM ${tableName}`); // MySQL only
```

New persistence:
- **MySQL** → extend `src/core/scripts/setupDatabase.ts` (run via
  `npm run db:setup`).
- **Supabase** → add a new timestamped file under `supabase/migrations/`
  (create the directory if it doesn't exist yet), applied via the
  Supabase dashboard SQL editor.

## Response envelope & errors (`src/core/types/index.ts`, `error.middleware.ts`)

```ts
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
  requestId?: string;   // auto-injected by requestLogger, don't set it yourself
}

export enum HttpStatus {
  OK = 200, CREATED = 201, BAD_REQUEST = 400, UNAUTHORIZED = 401,
  FORBIDDEN = 403, NOT_FOUND = 404, NOT_IMPLEMENTED = 501,
  INTERNAL_SERVER_ERROR = 500,
}
```

`ApiError` (`src/core/middlewares/error.middleware.ts`):

```ts
export class ApiError extends Error {
  statusCode: HttpStatus;
  constructor(statusCode: HttpStatus, message: string) { ... }
}
```

Controller pattern — always try/catch, always `next(error)`:

```ts
export const myAction = async (req: Request, res: Response, next: NextFunction) => {
  try {
    if (!valid) throw new ApiError(HttpStatus.BAD_REQUEST, 'reason');
    const response: ApiResponse<MyData> = { success: true, data: result };
    res.status(HttpStatus.OK).json(response);
  } catch (error) {
    next(error);
  }
};
```

The global `errorHandler` (mounted last in `src/index.ts`) catches
everything: `ApiError` → its own status+message; any other `Error` → 500
with the error's message; unknown throw → 500 generic message. Never
hand-format an error JSON response in a controller.

## Validation (`src/core/middlewares/validation.middleware.ts`)

Wraps `express-validator` chains and turns failures into a `400
ApiError` automatically:

```ts
// my-app.routes.ts — the actual pattern in the codebase
const helloWorldValidation = [
  body('firstName').notEmpty().withMessage('First name is required'),
];
router.post('/hello-world', validate(helloWorldValidation), helloWorld);
```

`validate()` also works with `param(...)` and `query(...)` chains (see
`apiLogs.routes.ts` for a `param('uuid').isUUID()` example).

## Config (`src/core/config/index.ts`)

One object, built once from `process.env` via `dotenv`, namespaced by
concern:

```ts
export const config = {
  port, environment, endpointPrefix,
  database: { type, tablePrefix, mysql: {...}, supabase: {...} },
  jwt: { secret, expiresIn, refreshExpiresIn },
  encryption: { masterKey },
  logging: { level },
};
```

Boot-fails immediately (`throw`) if `JWT_SECRET` or `MASTER_ENCRYPTION_KEY`
is missing from the environment. New env-driven settings should extend an
existing namespace or add a new one here — never read `process.env`
directly outside this file.

## Logging

`logger` (Winston, `src/core/utils/logger.ts`), level from
`config.logging.level` (`LOG_LEVEL` env var, default `info`).
