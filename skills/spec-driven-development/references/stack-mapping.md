# Stack Mapping

The SDD loop is fixed. The artefacts that fill each **role** differ by stack. Honor the
role; use the idiom. When you bootstrap or run the loop, pick the column for the project's
language and use those names and commands. If the project already has its own names for
these roles, defer to the project.

## Role → idiom

| Abstract role            | PHP / Laravel        | Python (Django)         | Python (FastAPI)        | Go                          | Node / TypeScript (Nest/Express) | Rust (Axum)            |
|--------------------------|----------------------|-------------------------|-------------------------|-----------------------------|----------------------------------|------------------------|
| Thin entry point         | Controller           | View                    | Path operation          | HTTP handler                | Controller / route handler       | Handler fn             |
| Input validation         | FormRequest          | Serializer / Form       | Pydantic request model  | Request struct + validator  | DTO + class-validator / zod      | Deserialize + validator|
| Single-purpose use case  | invokable Action     | Service / use-case fn   | Service / use-case fn   | Use-case func or struct     | Provider / use-case class        | Service fn             |
| Output shaping           | API Resource         | Serializer (read)       | Pydantic response model | Response DTO struct         | Serializer / interceptor         | Response struct + serde|
| Authorization            | Policy / Gate        | Permission class        | Dependency / guard      | Middleware / guard fn       | Guard                            | Extractor / middleware |
| Data access (DB-only)    | Eloquent + repository| ORM + repository        | SQLAlchemy + repository | sqlc / repository           | Prisma/TypeORM + repository      | sqlx / repository      |
| Domain events / async    | Event/Listener/Job   | Signal / Celery task    | BackgroundTasks / Celery| channel / goroutine / job   | EventEmitter / queue (BullMQ)    | tokio task / channel   |
| Migration                | Artisan migration    | Django migration        | Alembic                 | golang-migrate / goose      | Prisma migrate / TypeORM         | sqlx migrate / refinery|
| Test factory / fixture   | Model factory        | factory_boy / fixtures  | factory_boy / fixtures  | table-driven fixtures       | factory fn / fixtures            | builder fn             |

## Toolchain → command (formatter → refactorer → static analysis → tests)

Run these in step 6 (Refactor) and again before step 7 (Review/Smoke).

| Stack                | Format                | Static analysis        | Tests                          | Contract lint                    |
|----------------------|-----------------------|------------------------|--------------------------------|----------------------------------|
| PHP / Laravel        | `vendor/bin/pint`     | `vendor/bin/phpstan`   | `php artisan test --parallel`  | `npx @redocly/cli lint <spec>`   |
| Python               | `ruff format` + `ruff`| `mypy .`               | `pytest -q`                    | `npx @redocly/cli lint <spec>`   |
| Go                   | `gofmt -w .`          | `golangci-lint run`, `go vet ./...` | `go test ./...`   | `buf lint` (gRPC) / redocly      |
| Node / TypeScript    | `prettier -w .`       | `eslint .`, `tsc --noEmit` | `vitest run` / `jest`      | `npx @redocly/cli lint <spec>`   |
| Rust                 | `cargo fmt`           | `cargo clippy`         | `cargo test`                   | redocly / `buf lint`             |

Substitute the project's actual configured commands if they differ. Detect them by
reading `composer.json`, `pyproject.toml`/`Makefile`, `go.mod`+`Makefile`, `package.json`
scripts, or `Cargo.toml`.

## Test directory conventions per stack

| Layer        | Laravel                  | pytest                     | Go                       | Node/TS                  |
|--------------|--------------------------|----------------------------|--------------------------|--------------------------|
| Feature/API  | `tests/Feature/Api/V1/`  | `tests/feature/api/`       | `*_test.go` (handler)    | `test/e2e/` or `*.e2e.ts`|
| Unit         | `tests/Unit/`            | `tests/unit/`              | `*_test.go` (pkg)        | `*.spec.ts`              |
| Architecture | `tests/Architecture/`    | import-linter / custom     | `go vet` + custom        | `eslint` boundaries rule |
| Smoke        | `tests/Smoke/`           | `tests/smoke/`             | `tests/smoke/`           | `test/smoke/`            |

Architecture tests vary most. The intent is always the same: encode the layering rules as
an automated check (e.g. "entry points must not import the ORM directly", "use-cases must
not import the web framework"). Use whatever tool the ecosystem offers
(Pest arch plugin, `import-linter`, `go vet` + custom analyzers, `eslint-plugin-boundaries`).

## Frontend toolchain → command

For UI components/pages (see `references/frontend.md` for the full frontend loop).

| Stack               | Format        | Static analysis              | Tests                              | API client from contract     |
|---------------------|---------------|------------------------------|------------------------------------|------------------------------|
| React / TS          | `prettier -w .`| `eslint .` (+jsx-a11y), `tsc --noEmit` | `vitest run` + Playwright | `openapi-typescript` / `orval` |
| Vue / TS            | `prettier -w .`| `eslint .`, `vue-tsc`        | `vitest run` + Playwright          | `openapi-typescript` / `orval` |
| Svelte / SvelteKit  | `prettier -w .`| `eslint .`, `svelte-check`   | `vitest run` + Playwright          | `openapi-typescript`         |
| Angular             | `prettier -w .`| `ng lint`, `tsc`             | `jest`/`karma` + Playwright        | `openapi-generator`          |
| Server-rendered     | language formatter | template linter (if any) | framework feature/web tests        | n/a (server calls directly)  |

Accessibility checks belong in CI too: `jest-axe` in component tests and axe assertions in
Playwright. Visual regression (Playwright snapshots or Chromatic) is optional but
recommended for design-heavy work.

## Picking the right stack

If the repo is unambiguous (one manifest file, one language), use it. If it is polyglot
(e.g. a Go service plus a TypeScript frontend), each component runs its own copy of the
loop with its own column — the spec and ADRs are shared at the repo root, the tests and
toolchain are per-component. If the stack is unclear, ask once before generating anything.
