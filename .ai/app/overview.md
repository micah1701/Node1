# App Layer Overview

`src/app/` is the application-specific layer built on `src/core/`. It's
mounted in `src/app/routes/index.ts`:

```ts
router.use('/my-app', myAppRoutes);
```

...which is itself mounted at `config.endpointPrefix` (default `/api`)
alongside `coreRoutes` in `src/routes/index.ts`.

For full request/response field tables, see `src/app/README.md` — not
duplicated here.

## my-app — `/api/my-app`

Disposable node1 scaffold (`GET /status`, `POST /hello-world`) — safe to
remove or repurpose. It's also the reference template for adding a new
tool group; see [../skills/add-endpoint.md](../skills/add-endpoint.md).

This is the only thing mounted under `src/app/` in the base template. As
you build out your actual product, add new controllers/routes here (or a
sibling top-level dir under `src/`, like `src/notifications/` or
`src/lib/`, when something doesn't fit the controller/route shape) and
document their request/response fields in `src/app/README.md`. If a new
domain grows large enough to outgrow a table in that README, give it its
own file in this folder and link it from [../README.md](../README.md).
