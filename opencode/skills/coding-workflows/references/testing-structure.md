# Testing Structure — unit, integration, e2e (and the rest)

Use this whenever a workflow says "write tests" (Workflow 1 Step 5, Workflow 5 Step 4).
It is the lightweight twin of `spec-driven-development` Step 3. For red-first discipline
(tests must fail *before* code exists) and AC-mapped naming, use that skill.

## The test pyramid — pick the lowest layer that can express the check

Aim roughly for **70% unit / 25% integration / 5% e2e** (by count, not by line):

| Layer | What it tests | Speed | Real deps? | Example |
|-------|---------------|-------|------------|---------|
| **Unit** | Pure logic, one function/class | < 10ms | None — pure in/out | `validate.title('') === invalid` |
| **Integration** | Two+ layers wired, real DB/queue/cache | < 1s | Real DB (transactional rollback) | `POST /tasks → row in DB` |
| **Contract** | API contract matches reality | < 100ms | None (schema check) | `openapi-validate openapi.yaml` |
| **Smoke / e2e** | App boots + happy path + one failure path | < 10s total | Everything real | `curl /health → 200` |
| **Architecture** | Layering rules (handler ≠ DB driver) | < 50ms | None — static analysis | `deptrac analyse` |

**Rule:** every requirement/AC gets at least one test at the **lowest layer that can
express it**. Don't write an integration test for what a unit test can cover.

## What goes in each layer

- **Unit** — validators, mappers, calculators, serializers, pure business rules. No I/O, no DB, no network.
- **Integration** — the use-case end to end with a **real DB** (wrapped in a transaction that rolls back per test). Proves the layers wire together.
- **Contract** — if the change touches an HTTP API or a shared interface, assert the response shape matches the contract.
- **e2e / smoke** — one happy path + one failure path against the booted app. Keep it tiny; it's the slowest layer.

## AAA pattern (Arrange · Act · Assert)

Three visual blocks, blank lines between:

```js
test('creating a task without a title returns 422', async () => {
  // Arrange
  const payload = { /* no title */ }

  // Act
  const res = await request(app).post('/api/tasks').send(payload)

  // Assert
  assert.equal(res.status, 422)
  assert.match(res.body.error, /title/i)
})
```

**Hard rules:** one *act* per test · one *assertion concept* per test · **no conditionals**
(branching = two tests) · **no loops** (use parametrised `test.each` for cases).

## Naming — describe the observable outcome, not the function

Pick one style and never mix:

| Style | Example |
|-------|---------|
| should-when | `should_return_422_when_title_is_missing` |
| Given-When-Then | `given_no_title_when_creating_task_then_returns_422` |
| AC-mapped (if you have ACs) | `ac_003_creating_a_task_without_a_title_returns_422` |

## Isolation — every test is hermetic

- **No shared state** — each test gets a fresh DB transaction (rolled back), fresh store, or fresh container.
- **No ordering dependencies** — tests pass in any order, including run alone.
- **No real time** — inject a clock; freeze time in tests. Never `new Date()` in business logic.
- **No real randomness** — inject a seed. Never `Math.random()` in business logic.
- **No real network in unit/integration** — mock external HTTP at the boundary. Only e2e hits real services.

## Test data — factories beat fixtures

```js
function makeTask(overrides = {}) {
  return { id: 1, title: 'default title', done: false, ...overrides }
}
const task = makeTask({ title: 'something specific' })  // construct only what you care about
```

Use static fixtures only for reference data that rarely changes.

## Mocking discipline

**Mock:** external HTTP APIs (Stripe, SendGrid…), the clock, randomness, email/SMS/webhook
senders, filesystem writes in unit tests.

**Never mock:** your own code (the public interface IS the test), the DB in integration
tests (use a real DB with rollback), pure functions, standard library / framework code.

> If you're mocking your own service to test another of your services → write an
> integration test instead. Needing to mock internal code means the boundary is wrong.

## Coverage & speed budgets (floors, not targets)

| Metric | Target |
|--------|--------|
| Overall line coverage | ≥ 80% |
| Critical paths (auth, billing, data integrity) | 100% |
| Unit test | < 10ms each · full unit suite < 5s |
| Integration test | < 1s each · full suite < 60s |
| Flaky test | quarantine after 1 flake; fix or delete within the sprint |

100% coverage with weak assertions is worse than 80% with strong ones.

## Layout — mirror `src/` inside the test tree

```
tests/
├── unit/            # pure logic, no I/O   (mirrors src/)
├── integration/     # real DB, transactional rollback
├── contract/        # openapi.yaml ↔ implementation
├── e2e/             # smoke + happy + one failure path
└── factories/       # makeTask, makeUser, …
```

## Per-language cheat sheet

| Stack | Runner | Mocking | DB rollback |
|-------|--------|---------|-------------|
| **Jest / Vitest** | `jest` / `vitest` | `vi.mock` / `jest.mock` | `BEGIN; ROLLBACK` per test |
| **Node built-in** | `node --test` | `mock.method()` | same |
| **pytest** | `pytest` | `pytest-mock` | function-scoped transactional fixture |
| **PHPUnit / Pest** | `pest` / `phpunit` | `Mockery` | Laravel `RefreshDatabase` |
| **Go** | `go test` | interfaces + fakes | testcontainers + tx helper |
| **Rust** | `cargo test` | `mockall` | `sqlx::test` transactional rollback |

## Before you call it done

- [ ] Every requirement/AC has at least one test at the lowest sensible layer
- [ ] Unit + integration both present; e2e covers one happy + one failure path
- [ ] Test names describe outcomes; AAA blocks are visible
- [ ] No real time / randomness / network in unit + integration
- [ ] Factories exist for every entity created
- [ ] Security-sensitive behavior (auth, authz, validation, data exposure) is tested
- [ ] **Unit tests cover every changed unit of logic** — the base of the pyramid is never skipped
- [ ] Full suite is green
- [ ] **Coverage report run** and meets the budget (≥80% overall, 100% on critical paths) — a floor, not a target
