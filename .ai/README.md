# .ai/ — AI Assistant Documentation Index

This repo (`node1`, package name `typescript-api-framework`) is a base
template for building Node.js/TypeScript/Express REST APIs. It has two
layers, named the same way in the code and here:

- **`src/core/`** — the reusable node1 framework: auth, DB abstraction,
  encryption, key-value store, SSH key generation, request/audit logging.
- **`src/app/`** — where your actual product goes. Out of the box it only
  contains a disposable scaffold (`my-app`) that demonstrates the
  controller/route shape — delete or repurpose it when you build on this
  template.

Full endpoint request/response field tables live in `src/app/README.md`
(app layer) and the root `README.md` (core layer, framework install/extend
guide) — this folder doesn't duplicate those, to avoid drift. Read this
folder for architecture, conventions, and task recipes; read those READMEs
for exact field-by-field API contracts.

## Which file to read when

| If you're about to...                                       | Read |
|----------------------------------------------------------------|------|
| Make any change at all                                         | [global/rules.md](global/rules.md) |
| Get oriented / find where something lives                      | [global/project-overview.md](global/project-overview.md) |
| Check language/library versions, scripts, how to run           | [global/tech-stack.md](global/tech-stack.md) |
| Touch the DB layer, config, ApiResponse/error shape, validation | [core/patterns.md](core/patterns.md) |
| Touch auth, JWT, encryption, or request/audit logging          | [core/security.md](core/security.md) |
| Understand or extend the `src/app/` scaffold                    | [app/overview.md](app/overview.md) |
| Add a brand-new REST endpoint / tool group                      | [skills/add-endpoint.md](skills/add-endpoint.md) |

## Full file listing

- [global/rules.md](global/rules.md) — hard constraints and conventions
- [global/tech-stack.md](global/tech-stack.md) — languages, libraries, scripts, deployment
- [global/project-overview.md](global/project-overview.md) — architecture, directory map, request lifecycle
- [core/patterns.md](core/patterns.md) — DB abstraction, ApiResponse/HttpStatus, ApiError, config, validation
- [core/security.md](core/security.md) — JWT auth, encryption modes, audit logging/redaction
- [app/overview.md](app/overview.md) — the `src/app/` scaffold and how to extend it
- [skills/add-endpoint.md](skills/add-endpoint.md) — recipe for adding a new controller+route+registration

## Also relevant (outside .ai/, kept as source of truth, not duplicated)

- `README.md` (repo root) — node1 framework install/extend guide, core endpoint reference
- `src/app/README.md` — app-layer README; overwrite with your app's own
  endpoint reference once you build real features on the scaffold
