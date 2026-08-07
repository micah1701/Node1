# Skill: Add a New Endpoint / Tool Group

Recipe for adding a new REST endpoint in `src/app/`, using the `my-app`
scaffold as the worked template.

## 1. Controller

`src/app/controllers/<name>.controller.ts`:

```ts
import { Request, Response, NextFunction } from 'express';
import { HttpStatus, ApiResponse } from '../../core/types';
import { ApiError } from '../../core/middlewares/error.middleware';

export const myAction = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { firstName, lastName } = req.body;
    if (!firstName || !lastName) {
      throw new ApiError(HttpStatus.BAD_REQUEST, 'firstName and lastName are required');
    }

    const response: ApiResponse<{ someField: string }> = {
      success: true,
      data: { someField: 'someValue' },
      message: `App created for ${firstName} ${lastName}`,
    };
    res.status(HttpStatus.OK).json(response);
  } catch (error) {
    next(error);
  }
};
```

(Verbatim shape of `src/app/controllers/my-app.controller.ts`.)

## 2. Route file

`src/app/routes/<name>.routes.ts`:

```ts
import { Router } from 'express';
import { body } from 'express-validator';
import { validate } from '../../core/middlewares/validation.middleware';
import { authenticate } from '../../core/middlewares/auth.middleware';
import { myAction } from '../controllers/<name>.controller';

const myRoutes = Router();

const myActionValidation = [
  body('firstName').notEmpty().withMessage('First name is required'),
  body('lastName').notEmpty().withMessage('Last name is required'),
];

myRoutes.post('/my-action', authenticate, validate(myActionValidation), myAction);

export default myRoutes;
```

Add `authenticate` (and `authorize([roles])` after it) only if the
endpoint should require a JWT — see [../global/rules.md](../global/rules.md).

## 3. Register in the app router

`src/app/routes/index.ts`:

```ts
import myRoutes from './my.routes';
// ...
router.use('/my', myRoutes);   // → mounted at /api/my/...
```

## 4. New config (if needed)

Add a namespaced block to `src/core/config/index.ts` — don't read
`process.env` in the controller. See [../core/patterns.md](../core/patterns.md).

## 5. New persistence (if needed)

- MySQL path → extend `src/core/scripts/setupDatabase.ts`.
- Supabase path → new file under `supabase/migrations/`.

See [../core/patterns.md](../core/patterns.md) for the `DatabaseInterface`
pattern.

## 6. Document it

- Add the endpoint's request/response fields to `src/app/README.md` (the
  maintained source of truth for field-level API contracts).
- If it's a new top-level domain (not just a new action on an existing
  one), add a short paragraph to [../app/overview.md](../app/overview.md)
  and a row to the [../README.md](../README.md) index if it warrants its
  own file.
