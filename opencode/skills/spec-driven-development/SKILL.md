---
name: spec-driven-development
description: >
  Spec-driven development (SDD) workflow for any language or framework — PHP/Laravel,
  Python/Django/FastAPI, Go, Node/TypeScript, Rust, and others. Use this whenever
  building, adding, or changing a feature in a project that follows (or should follow)
  a spec-first discipline: write a spec, define a contract, write failing tests, then
  implement. Trigger when the user wants to write a spec or an ADR, set up / bootstrap
  SDD in a repo, add numbered acceptance criteria, or implement a feature "the right
  way" with traceability from intent to code. Also trigger for any database-backed
  feature (migrations, repositories, search) where validation, authorization, and
  data-exposure rules must be pinned down before coding. Applies equally to backend,
  frontend (React/Vue/Svelte/Angular or server-rendered views), and full-stack features —
  the frontend consumes the same contract. Do not wait for the literal words
  "spec-driven development" — a request to add a feature in a repo that has a docs/specs
  or docs/adr folder is enough.
compatibility: opencode
---

# Spec-Driven Development (SDD)

> **Task:** $ARGUMENTS

---

## Step −1 — Which flow do you want to run?

Before SDD-specific questions, ask the user **which overall flow** fits the task.
This skill is most powerful when chained with sibling skills.

Ask the user this question and wait for their reply: **"Which flow fits this task?"** (single-select — pick exactly one):

1. **Clarify only** — Just produce numbered ACs (clarify-loop). Use when you'll implement later or hand off.
2. **Brainstorm → design** — Structured brainstorming → writing-plans. Use when the idea is fuzzy and needs shape first.
3. **Quick build (simple code flow)** — coding-workflows lightweight flow — no spec/contract. Use for trivial changes with no contract or data impact.
4. **Plan → Build → Verify (PBV)** — coding-workflows Workflow 5: plan + human approval → small tasks → multi-agent build → tests → smoke/e2e → lint → human review. Use when the change deserves a plan but not a frozen spec.
5. **Full SDD (Recommended)** — spec → contract → red tests → implement → smoke. Use when data, auth, or API contract is touched.
6. **Full SDD — tests after (no TDD)** — coding-workflows Workflow 6: same SDD rigor (spec, contract, ACs, validation, DoD) but tests are written AFTER implementation and must pass — not red-first. Use when you want SDD discipline without test-first.
7. **Full SDD + TDD** — SDD + a strict red-first TDD loop. Use for high-risk or regulatory work — strictest mode.
8. **Debug existing bug** — systematic debugging method. Use when this is a bug, not a feature.
9. **Architecture only** — coding-workflows architecture flow + ADR. Use when you need a decision, not implementation.

Store the selected label as `<flow>`. If the user picks anything other than **Full SDD** or **Full SDD + TDD**, hand off to the corresponding skill and exit. Otherwise continue with Step 0 below.

### Routing table — every flow maps to specific skills

| Picked flow | First skill | Then | Final |
|-------------|-------------|------|-------|
| **Clarify only** | `ai-coding-toolkit:clarify-loop` | — | stop after ACs |
| **Brainstorm → design** | structured brainstorming | writing an implementation plan | hand to user |
| **Quick build (simple code flow)** | `ai-coding-toolkit:clarify-loop` (sections 1, 2 only) | `ai-coding-toolkit:coding-workflows` (Workflow 1) | a verification-before-completion checklist |
| **Plan → Build → Verify (PBV)** | `ai-coding-toolkit:clarify-loop` (sections 1, 2) | `ai-coding-toolkit:coding-workflows` (Workflow 5) + `ai-coding-toolkit:multi-agent` | verification-before-completion checklist + an adversarial AI code review |
| **Full SDD** | `ai-coding-toolkit:clarify-loop` (all sections) | this SDD skill (Step 0 → 8) | an adversarial AI code review |
| **Full SDD — tests after (no TDD)** | `ai-coding-toolkit:clarify-loop` (all sections) | `ai-coding-toolkit:coding-workflows` (Workflow 6) — runs SDD steps in test-after order | an adversarial AI code review |
| **Full SDD + TDD** | `ai-coding-toolkit:clarify-loop` | a strict red-first TDD loop for tests + this SDD skill for everything else | an adversarial AI code review + verification-before-completion checklist |
| **Debug existing bug** | systematic debugging method | `ai-coding-toolkit:coding-workflows` (Workflow 2) for regression test | verification-before-completion checklist |
| **Architecture only** | `ai-coding-toolkit:coding-workflows` (Workflow 3) | this SDD skill (Step 8 ADR only) | hand to user |

If the user picks anything other than **Full SDD** or **Full SDD + TDD**, hand off and exit this skill.

---

A process for shipping every meaningful change — from a one-line fix to a new
feature — through the same loop, so that **intent, contract, validation, security,
tests, and code never drift apart**. A year later, anyone reading the repo can answer
*why* a line exists, not just *what* it does.

---

## Step 0 — Ask the user which steps to run

Before running any step, ask the user **two questions** and wait for answers.

### Question 1 — Which steps do you want to include?

Ask the user this question and wait for their reply: **"Which SDD steps should I run for this task?"** (multi-select — pick any number):

1. **1. Spec — docs/specs/NNN-feature.md with numbered ACs** — The intent layer — frozen once a test references it.
2. **2. Contract — update docs/api/openapi.yaml** — HTTP features only. Skip for non-HTTP work.
3. **3. Failing tests (red first)** — Tests written before implementation, mapped to ACs.
4. **4. Validation + Security checklist** — Input rules, auth, ownership, data exposure — pin down before coding.
5. **5. Implement — minimum code to make tests green** — YAGNI. Sequential or parallel agents.
6. **6. Refactor — formatter, linter, static analyzer, full test suite** — Clean up while green. Tools chosen in Question 2.
7. **7. Review + Smoke test** — Human + AI review, generated smoke script, auto-fix loop on failures.
8. **8. ADR — record non-obvious decisions** — Skip if no decision worth recording.

**Sensible skip combinations:**
- Non-HTTP feature: skip 2 (Contract)
- Trivial bug fix: skip 2, 8 (Contract + ADR)
- Documentation-only change: skip 2, 3, 5 (Contract, Tests, Implement)
- Bootstrapping SDD in a fresh repo: run all + scaffold templates first

Store selected steps as `<active-steps>`. Skip any step not in this set.

### Question 2 — Which lint/test tools should the Refactor step run?

**First detect** what's available in the project (scan package.json scripts, composer.json scripts, Cargo.toml, go.mod, Makefile, justfile). Then ask the user this question and wait for their reply: **"Which tools should run in Step 6 (Refactor)?"** (multi-select — pick any number), presenting the detected tools:

1. **Formatter — {{detected: prettier/pint/black/gofmt/…}}** — Auto-format code. Fastest, never breaks anything.
2. **Auto-refactor — {{detected: rector/ts-morph/autoflake/…}}** — Apply mechanical refactors. Skip if N/A.
3. **Linter — {{detected: eslint/phpstan/ruff/golangci-lint/clippy}}** — Catch style + correctness issues.
4. **Type checker — {{detected: tsc/mypy/phpstan/cargo check}}** — Verify types. Slowest but most valuable.
5. **Full test suite — {{detected: npm test/pytest/composer test/go test ./…}}** — Run every test in the repo.
6. **Smoke test — generated in Step 7** — End-to-end check via curl against running app.

Any tools the user wants to add get appended to `<active-tools>`.

Store selected tools as `<active-tools>`. Step 6 runs only these in order.

---

> The system should say what it means, test what it promises, and implement only what
> was agreed.

This skill is **language- and framework-agnostic**. The loop is fixed; the tools are
whatever the target project uses. Before writing anything, find out (or ask) what stack
the project is, then map the abstract roles below onto that stack — see
`references/stack-mapping.md`.

---

## The Loop

```
1. Spec → 2. Contract → 3. Failing Tests → 4. Validation+Security
   → 5. Implement → 6. Refactor → 7. Review+Smoke → 8. ADR (if any)
   ↑__________ re-spec when a decision invalidates the original intent __________|
```

Steps 2 (contract) and 8 (ADR) are conditional. Everything else always happens, even
for small changes. Trivial fixes may collapse the spec into a referenced bug/issue, but
they still get a failing test first.

Full step-by-step detail, including the validation and security checklists, lives in
`references/workflow.md`. Read it before running the loop for the first time in a
session. The condensed version follows.

### 1. Spec — *what and why, before any code*
Write `docs/specs/NNN-feature-name.md` from `assets/spec-template.md`. Required sections:
Goal, User stories, **numbered** Acceptance Criteria (AC-001, AC-002, …), Endpoints (if
HTTP-facing), Validation rules, Authorization rules, Domain rules, Edge cases, Security
concerns, Out of scope. ACs must be observable, testable, specific, and small enough to
map to one or more tests. A spec is **frozen once a test references it** — after that,
change it only via a dated amendment block or a new superseding spec, never a silent
edit.

#### 1a. Spec approval gate — MANDATORY before continuing

After writing the spec, **stop and get explicit user approval before anything else**.
The spec is the source of truth for every subsequent step; a wrong spec cascades into
wrong contracts, wrong tests, and wrong code.

First show the spec file path and the AC list inline, then ask the user this question and wait for their reply: **"Spec written to docs/specs/NNN-feature-name.md with {{N}} ACs. Approve?"** (single-select — pick exactly one):

1. **Approve — proceed to contract / tests** — Lock the spec. From here, changes require a dated amendment block.
2. **Edit ACs — change specific acceptance criteria** — I'll ask which ACs to change, apply the edits, then re-ask for approval.
3. **Rewrite section — Goal / Validation / Auth / Edge cases / Out of scope** — I'll ask which section, rewrite it, then re-ask for approval.
4. **Run AI review first (an adversarial AI code review)** — Adversarial review catches vague, untestable, or overlapping ACs before you approve.
5. **Reject — go back to clarify-loop** — The spec doesn't match my intent. Restart clarification with what's missing.

**Behavior per choice:**
- **Approve** → mark the spec as frozen, write `Frozen: <date>` in the spec footer, continue to Step 2
- **Edit ACs** → ask the user which AC numbers to change, apply changes, loop back to 1a
- **Rewrite section** → ask the user which section, rewrite, loop back to 1a
- **AI review** → run an adversarial AI code review on the spec, surface findings, loop back to 1a
- **Reject** → exit SDD, hand off to `ai-coding-toolkit:clarify-loop` with the original task

Do **not** proceed to Step 2 until the user picks Approve.

### 2. Contract — *machine-readable counterpart of the spec* (HTTP features only)
Update the API contract (`docs/api/openapi.yaml`, or the project's gRPC/GraphQL/proto
equivalent) **before** application code. Prose and contract must agree. Every endpoint
gets explicit request/response schemas, at least one success and one error response, and
validation that matches the application's real rules. Lint it in CI. Non-HTTP changes
skip this step but the spec must say so explicitly.

### 3. Failing Tests — *red first*
Write tests against the spec and contract, not against code that does not yet exist.
They must fail first. This is the first validation checkpoint: **if you cannot write a
clear test, the spec is too vague.** Name tests after acceptance criteria where practical
(e.g. `ac_003_creating_a_post_without_a_title_returns_422`). Security-sensitive behavior
*must* have tests.

#### 3a. Test layers and the pyramid

Aim roughly for **70% unit / 25% integration / 5% e2e** (by count, not by line):

| Layer | What it tests | Speed | Real deps? | Example |
|-------|---------------|-------|------------|---------|
| **Unit** | Pure logic, one function/class | < 10ms | None — pure inputs/outputs | `validate.title('').valid === false` |
| **Integration** | Two or more layers wired together, real DB/queue/cache | < 1s | Real DB (transactional), real ORM | `POST /api/tasks → row in DB` |
| **Contract** | The API contract matches reality | < 100ms | None (schema check) | `openapi-validate openapi.yaml` |
| **Smoke / e2e** | App boots + happy path + one failure path | < 10s total | Everything real | `curl /health → 200` |
| **Architecture** | Layering rules (handlers don't import the DB driver) | < 50ms | None — static analysis | `deptrac analyse` |

**Rule:** every AC must have at least one test at the **lowest layer that can express it**.
Don't write an integration test for what a unit test can cover.

#### 3b. AAA pattern (Arrange · Act · Assert)

Every test follows this shape — three visual blocks, blank lines between:

```js
test('ac_002 creating a task without title returns 422', async () => {
  // Arrange
  const payload = { /* no title */ }

  // Act
  const res = await request(app).post('/api/tasks').send(payload)

  // Assert
  assert.equal(res.status, 422)
  assert.match(res.body.error, /title/i)
})
```

**Hard rules:**
- One **act** per test (multiple acts = multiple tests)
- One **assertion concept** per test — multiple `assert.*` lines on the *same concept* are fine; assertions across unrelated concepts are not
- **No conditionals** in tests (`if/else`, ternaries) — branching is a sign you need two tests
- **No loops** in tests — except for parametrised cases via a list and `test.each`

#### 3c. Naming convention

Use one of these consistently across the repo:

| Style | Example |
|-------|---------|
| AC-mapped (preferred for SDD) | `ac_003_creating_a_post_without_a_title_returns_422` |
| Given-When-Then | `given_no_title_when_creating_task_then_returns_422` |
| should-when | `should_return_422_when_title_is_missing` |

Pick one and never mix. The test name **must** describe the observable outcome — not the function name.

#### 3d. Isolation — every test is hermetic

- **No shared state between tests.** Each test gets a fresh DB transaction (rolled back), a fresh in-memory store, or a fresh container.
- **No test ordering dependencies.** Tests must pass when run in any order, including alone (`test --filter`).
- **No real time.** Inject a clock; never `new Date()` inside business logic. Tests freeze time.
- **No real randomness.** Inject a seed; never `Math.random()` inside business logic.
- **No real network in unit/integration.** Mock external HTTP at the boundary. Smoke/e2e is the only layer allowed to hit real services.

#### 3e. Test data — factories beat fixtures

- **Factories** (functions that build objects with sensible defaults + overrides) — preferred
- **Fixtures** (static JSON/YAML files) — only for reference data that rarely changes

```js
// Good — factory
function makeTask(overrides = {}) {
  return { id: 1, title: 'default title', done: false, created_at: '2026-01-01', ...overrides }
}

// Then in tests:
const task = makeTask({ title: 'something specific' })
```

Each test should construct **only the data it cares about**. Defaults handle the rest.

#### 3f. Mocking discipline — when yes, when no

**Mock these:**
- External HTTP APIs (Stripe, SendGrid, third-party services)
- The clock (frozen time)
- Randomness (seeded RNG)
- Email senders, SMS senders, webhook senders
- File system writes in unit tests (use in-memory FS)

**Never mock these:**
- Your own code (use real implementations — the public interface IS the test)
- The database in **integration tests** (use a real DB with rollback)
- Pure functions (just call them)
- Standard library / framework code

**If you find yourself mocking your own service to test another of your services → integration test it instead.** The need to mock internal code is a smell that the boundary is wrong.

#### 3g. Coverage and speed budgets

| Metric | Target |
|--------|--------|
| Overall line coverage | ≥ 80% |
| Critical paths (auth, billing, data integrity) | 100% |
| Unit test runtime | < 10ms each, full unit suite < 5s |
| Integration test runtime | < 1s each, full integration suite < 60s |
| Smoke runtime | < 10s total |
| Flaky test policy | quarantine after 1 flake; delete or fix within the sprint |

Coverage is a **floor**, not a target. 100% coverage with weak assertions is worse than 80% with strong assertions.

#### 3h. Other patterns to use when they fit

- **Snapshot/golden tests** — for output serializers, generated SQL, API response shapes. Update intentionally, never blindly.
- **Property-based tests** — for validators, parsers, encoders. Generate 100s of inputs to find edge cases.
- **Contract tests** — for any consumer-producer pair (e.g. frontend ↔ backend). Run on both sides against the same `openapi.yaml`.
- **Mutation testing** — periodically run a mutator (Stryker, mutmut) to verify your tests actually catch bugs.

#### 3i. Per-language cheat sheet

| Stack | Test runner | Mocking | DB rollback / isolation | Snapshot | Property-based |
|-------|-------------|---------|-------------------------|----------|----------------|
| **Node (built-in)** | `node --test` | `mock.method()` | `BEGIN; ROLLBACK` per test | `assert.snapshot` (3rd-party) | `fast-check` |
| **Jest / Vitest** | `jest` / `vitest` | `vi.mock` / `jest.mock` | Same | `toMatchSnapshot()` | `fast-check` |
| **pytest** | `pytest` | `pytest-mock` / `unittest.mock` | `@pytest.fixture(scope='function')` + transactional rollback | `pytest-snapshot` | `hypothesis` |
| **PHPUnit / Pest** | `pest` / `phpunit` | `Mockery` / `prophecy` | Laravel `RefreshDatabase` trait | `pest-plugin-snapshots` | `eris` |
| **Go** | `go test` | interfaces + small fakes | testcontainers + transactional helper | `cupaloy` | `gopter` |
| **Rust** | `cargo test` | `mockall` / trait-based fakes | `sqlx::test` with transactional rollback | `insta` | `proptest` |

#### 3j. Test naming and layout

```
tests/
├── unit/                    # pure logic, no I/O
│   ├── validate.test.js
│   └── serialize.test.js
├── integration/             # real DB / real queues, transactional rollback
│   ├── tasks.create.test.js
│   └── tasks.list.test.js
├── contract/                # openapi.yaml ↔ implementation
│   └── openapi.test.js
├── e2e/                     # smoke + happy + one failure path
│   └── tasks.flow.test.js
└── factories/               # makeTask, makeUser, etc.
    └── task.js
```

Mirror the `src/` structure inside `tests/unit/` and `tests/integration/`.

#### 3k. The red-first checklist before writing code

Before moving to Step 4 (Validation+Security), confirm:

- [ ] Every AC has at least one test, at the lowest sensible layer
- [ ] All tests **fail** when run now (red first)
- [ ] Test names map back to AC IDs
- [ ] Factories exist for every entity you create
- [ ] No real time, randomness, or network in unit/integration tests
- [ ] Security-sensitive behavior (auth, authz, validation, data exposure) has tests

### 4. Validation & Security — *part of the contract, not cleanup*
Before implementing, answer the validation checklist (required/optional fields, types,
format, length, range, allowed values, cross-field rules, uniqueness, failure response)
and the security checklist (authn, authz, ownership, roles, data exposure, mass
assignment, rate limiting, error leakage, auditability, abuse cases). Put each concern in
its correct layer (see placement table in `references/workflow.md`) — never rely on the
entry-point handler to "remember" rules.

### 5. Implement — *minimum code to make tests green*

#### 5a. Ask: sequential or parallel?

```
How do you want to implement this? (default: 2)

  1. Sequential  — one layer at a time in this context (safe, simple)
  2. Parallel    — fan out layers to parallel agents (faster, needs layer isolation)

For parallel: each agent owns exactly one file/layer so there are no conflicts.
If an MCP swarm server is configured it is used; otherwise spawn subagents with
opencode's `task` tool, one file per agent.
```

Store as `<impl-mode>`.

#### 5b-seq. Sequential implementation (if mode = 1)

Default order: schema/migration → entity/model → test factory/fixture → authorization
rule → use-case/action → input validator → output serializer → handler/endpoint → route.
Stop when green. **YAGNI is load-bearing**: add no field, endpoint, abstraction, helper,
permission, or behavior the spec did not ask for. Keep entry points thin.

#### 5b-par. Parallel agent implementation (if mode = 2)

**Pre-flight: assign one file per agent — no two agents share a file.**

Map the spec's layers to agents. Typical split for an HTTP feature:

| Agent | Layer | Owns |
|-------|-------|------|
| A | Schema / Migration | `database/migrations/NNN_*.sql` or ORM migration file |
| B | Model / Entity | `src/models/FeatureName.js` (or equivalent) |
| C | Validator / Input | `src/validators/featureName.js` |
| D | Use-case / Action | `src/actions/FeatureName.js` |
| E | Serializer / DTO | `src/serializers/featureName.js` |
| F | Handler + Route | `src/handlers/featureName.js` + route registration |

Adjust to the project's actual layer names (see `references/stack-mapping.md`).

**If an MCP swarm server is configured**, use it to spawn one agent per role
(schema, model, validator, action, serializer, handler), each tasked to implement
its single layer for `<feature>` per `docs/specs/NNN.md`.

**Otherwise — spawn subagents with opencode's `task` tool, one file per agent (send all in ONE message):**

Spawn all agents in a single message with parallel `task` tool calls. Each agent prompt must include:
- The spec file path and the specific AC numbers it is responsible for
- The exact file it must write (one file only)
- The layer contract: what it receives as input, what it must return
- The YAGNI constraint: implement only what the spec requires
- Instruction to return the full file content as its response

**Conflict detection before spawning:**
List all files each agent will write. If any file appears twice → serialize those two agents (run the second after the first completes). Never let two agents write the same file.

**After all agents complete:**

1. **Merge** — collect all agent outputs into the working tree
2. **Wire** — ensure layers import each other correctly (handler imports action, action imports validator, etc.)
3. **Run tests** — `<test-command>` from Step 0. Show result:
   ```
   Post-merge test run:
     ✔ 6/6 tests pass
     — or —
     ✘ 2 failures (agent C validator missing required field check — fix inline)
   ```
4. **Fix inline** if any test fails — do not re-spawn agents for small fixes; fix in this context and re-run
5. Proceed to Step 6 (Refactor) only when all tests are green

### 6. Refactor — *clean up while green*
Run **only the tools the user selected in `<active-tools>` (Step 0, Question 2)** —
in this order: formatter → auto-refactorer → linter → type checker → full test suite →
smoke test. Show each command and its result with a checklist as you go:

```
Refactor pass (running <active-tools>):
  ✔ pint           → 0 issues
  ✔ rector         → 0 changes
  ◼ phpstan        → 2 warnings (showing)
  ⏳ composer test  → running
  ⏳ smoke test     → pending
```

Remove duplication, improve names, simplify flow, clarify boundaries, delete dead code.
The public contract must not change here unless the spec and contract were updated
first. Refactor only while the suite stays green. **If a tool fails, stop and report —
do not continue to the next tool.**

### 7. Review & Smoke — *green ≠ mergeable*

#### 7a. Ask the user which review modes to run

Ask the user this question and wait for their reply: **"How do you want to review this change?"** (multi-select — pick any number):

1. **Human review** — I'll show the diff and pause until you reply 'approved' or list changes. Catches what tests can't.
2. **AI review (an adversarial AI code review)** — Adversarial reviewer tries to find logic bugs, security holes, performance issues. Applies fixes per your call.
3. **Smoke test (generated script + auto-fix loop)** — Generate scripts/smoke/<feature>.sh, run it, drop failures into the debug loop until green.
4. **Verification checklist (a verification-before-completion checklist)** — Forces evidence before any 'done' claim — test output, lint pass, smoke output.

Store the selected labels as `<review-mode>`. Run each selected check in order: Human → AI → Smoke → Verification. Skip any not selected.

#### 7b. Human review (if selected)

First, show the diff inline (`git diff` for unstaged work, or `git diff main...HEAD` for branch work) — keep it focused on the changed files only.

Then ask the user this question and wait for their reply: **"Implementation diff above. Approve? Check: spec match · contract match · AC coverage · validation/authz explicit · no sensitive data exposed · no out-of-spec behavior"** (single-select — pick exactly one):

1. **Approve — proceed to AI / smoke / next step** — Diff matches spec and intent. Continue.
2. **Request changes** — I'll ask which files/lines, apply the changes, then re-show the diff.
3. **Run AI review first, then decide** — Get an adversarial second opinion before approving. Loops back to this question after.
4. **Reject — return to Step 5 (Implement)** — Diff doesn't match spec. Restart implementation with what's missing.

If "Request changes": ask for the change list via free text, apply, re-show diff, loop back to 7b. Only proceed when the user picks **Approve**.

#### 7c. AI review (if selected)

Run an adversarial AI code review on the diff. That review spawns an adversarial reviewer that:
- Tries to find logic bugs and edge cases
- Tries to find security holes (injection, auth bypass, data leaks)
- Tries to find performance issues (N+1, blocking calls)
- Checks the diff against the spec and contract

For each finding the AI reviewer produces, ask the user this question and wait for their reply: **"AI reviewer finding: {{title}} — {{severity}}. Detail: {{one-line summary}}"** (single-select — pick exactly one):

1. **Apply — fix now** — Apply the suggested fix, re-run the relevant tests.
2. **Defer — track as follow-up issue** — Log to TODO/issue tracker with rationale. Don't block this PR.
3. **Reject — reviewer is wrong** — Document why in PR notes. If the spec caused the confusion, amend it.
4. **Show full detail before deciding** — Print the full reviewer rationale + diff suggestion, then re-ask.

Loop until the AI reviewer returns no new high-severity findings. Track all decisions in a single summary at the end (e.g. "Applied 3, deferred 1, rejected 1").

#### 7d. Smoke test (if selected)

Human review confirms the implementation matches the spec, the contract matches real
behavior, ACs are covered, validation/authorization/ownership are explicit and tested,
sensitive data is not exposed, and no out-of-spec behavior crept in.

#### Generate and run a smoke test script

After review, **generate a real smoke test script** rather than running ad-hoc commands.

1. **Write the script** to `scripts/smoke/<feature-slug>.sh` (or the project's
   equivalent — `.ps1`, `.py`, `.mjs`). It must cover:
   - App boots and listens on the expected port
   - Routes for the feature are registered (list / curl `--head`)
   - Database / queue / cache is reachable
   - Auth works (login / token exchange)
   - The happy path of the changed endpoint returns 2xx with the contract shape
   - At least one failure path returns the correct 4xx with the contract error shape
   - Sensitive fields are NOT in the response

   **Portability rules (macOS + Linux compatible):**
   - Never use `head -n -1` (BSD `head` does not support negative line counts)
   - Capture body and status separately: `BODY=$(curl -s -o /tmp/smoke_body.json -w "%{http_code}" ...)` then `STATUS=$?` and `cat /tmp/smoke_body.json`
   - Never use `date -d` (GNU only) — use `date -u +%Y-%m-%dT%H:%M:%SZ` for timestamps
   - Never use `grep -P` (PCRE) — use `grep -E` (ERE) which works on both BSD and GNU
   - Use `command -v` instead of `which` to check for tool presence
   - Use `mktemp` for temp files, not hardcoded `/tmp/` paths

2. **Run the script** and capture output. Present results as a checklist:

   ```
   Smoke test: docs/specs/001-post-bookmarks.md
     ✔ app boots (8000)            200ms
     ✔ POST /api/bookmarks routed
     ✔ db reachable
     ✔ login → token
     ✔ happy: POST /api/bookmarks → 201 (shape matches contract)
     ✘ fail: POST /api/bookmarks (no title) → got 500, expected 422
     ✔ no `password_hash` in response

   1 failure. Dropping into debug loop.
   ```

3. **If any check fails → enter the debug & fix loop** from the `coding-workflows`
   skill (Workflow 2: Debugging — Reproduce → Read error → Isolate layer →
   Hypothesize → Fix, verify, prevent). For each failure:
   - State the symptom in one line
   - Read the relevant log lines / stack trace
   - Ask: which layer owns this — handler, validator, action, repository, db?
   - Apply the minimum fix
   - Re-run **only the failing smoke check first**, then the full script
   - Loop until all checks pass

4. **Add a regression test** in the unit/feature suite for every smoke failure
   before marking complete. A bug that came back once will come back again.

5. **Final review gate after fixes** — when all smoke checks pass, ask the user this question and wait for their reply: **"All smoke checks pass after fixes. Which final reviews should I run?"** (multi-select — pick any number):

   1. **Human review of the fix diff** — Show diff of files touched during smoke fixes. Catches regressions introduced while fixing.
   2. **AI review (an adversarial AI code review) on the fix** — Adversarial review focused on the fix diff. Each finding gets an Apply/Defer/Reject decision.
   3. **Verification checklist (a verification-before-completion checklist)** — MANDATORY before any 'done' claim — forces test output, lint pass, smoke output as evidence.
   4. **Re-run full test suite + smoke** — Belt-and-braces final check before marking complete.

   A verification-before-completion checklist is strongly recommended — it forces
   evidence (test output, smoke output, lint pass) before any success claim, commit, or PR.

If smoke fails for a reason that is actually a spec gap (the test is right, the spec
is incomplete) — return to step 1 (Spec) and amend, do not patch around it.

### 8. ADR — *record non-obvious decisions* (when warranted)
Write `docs/adr/NNNN-title.md` from `assets/adr-template.md` for decisions like: choosing
a dependency, a column type, a new architectural pattern, rejecting a likely option, a
simplicity/performance/flexibility tradeoff, a security or authz change, an API-versioning
change, or choosing queue/cache/search/storage infra. **ADRs are immutable once merged** —
a later decision creates a new ADR that supersedes the old one by number, never edits it.

---

## Two ways this skill is used

### A) Bootstrap SDD into a project
The repo has no SDD scaffolding yet, or the user says "set this up." Create the layout
and seed it with the templates, adapted to the detected stack:

```
docs/
├── sdd/00-workflow.md        # copy of references/workflow.md, stack-adapted
├── specs/                    # NNN-feature.md, one per feature
├── adr/                      # NNNN-title.md, immutable decision records
├── api/openapi.yaml          # contract (HTTP projects only)
├── conventions.md            # from assets/conventions-template.md, filled in
└── glossary.md               # from assets/glossary-template.md, filled in
```

Fill `conventions.md` and `glossary.md` with the project's **actual** language version,
tools, layer names, and domain terms — do not leave Laravel/PHP placeholders in a Python
or Go repo. Use `references/stack-mapping.md` to translate.

### B) Run the loop for a feature
The scaffolding exists and the user wants a feature. Walk the eight steps in order,
producing the artefacts in sequence (spec file exists *before* the test file is
committed). Keep the user in the loop at the spec and contract gates — those are where
intent is cheapest to correct. Surface the Definition of Done (below) at the end.

---

## Adapting to the stack — the core idea

The original of this workflow was written for Laravel. Every Laravel-specific noun maps
to an abstract **role**. Honor the role; use the stack's idiom for it.

| Abstract role            | Laravel/PHP        | Python (Django/FastAPI)      | Go                        |
|--------------------------|--------------------|------------------------------|---------------------------|
| Thin entry point         | Controller         | View / path operation        | HTTP handler              |
| Input validation         | FormRequest        | Serializer / Pydantic model  | Request struct + validate |
| Single-purpose use case  | invokable Action   | Use-case / service function  | Use-case function/struct  |
| Output shaping           | API Resource       | Serializer / response model  | DTO / response struct     |
| Authorization            | Policy             | Permission class / dependency| Middleware / guard fn     |
| Data access (DB only)    | Eloquent + repo    | ORM + repository             | sqlc / repository         |
| Domain events / async    | Event/Listener/Job | Signals / Celery task        | channel / goroutine / job |
| Formatter / linter / SA  | Pint/Rector/PHPStan| ruff/black + mypy            | gofmt/golangci-lint + vet |
| Test runner              | Pest/PHPUnit       | pytest                       | go test                   |

Full mapping (including Node/TypeScript and Rust), with concrete toolchain commands per
stack, is in `references/stack-mapping.md`. Database-specific guidance — repository
isolation, migration safety, transaction boundaries, DB-gated tests, full-text search —
is in `references/database.md`; read it for any data-backed feature.

**Frontend work runs the same loop.** The frontend does not own a contract — it *consumes*
the backend's (`docs/api/openapi.yaml`), generating a typed client from it so the two
sides cannot drift. Three things become first-class acceptance criteria on the client:
every **UI state** (loading/empty/error/partial/success), **accessibility** (keyboard,
focus, labels, contrast), and the rule that **client validation is UX, never the security
boundary** — the server stays the only trusted side. The full frontend mapping (roles,
test layers, security) is in `references/frontend.md`; use `assets/component-spec-template.md`
for UI specs.

If the project's stack is unclear, **ask once** which language/framework and test runner
it uses, then proceed. Do not invent a toolchain.

---

## Definition of Done

A change is mergeable only when:

- The spec exists and reflects intended behavior; ACs are numbered and testable.
- The contract is updated and matches real behavior (HTTP features), or the spec states
  the contract is unaffected.
- Tests exist, map back to ACs, and **failed before** implementation.
- Input validation is explicit and tested.
- Authentication, authorization, and ownership rules are explicit and tested.
- Sensitive data is not exposed; error responses are intentional and documented.
- Formatting, static analysis, the full test suite, and smoke tests all pass.
- The PR has been reviewed; any non-obvious decision has an ADR.
- No behavior outside the spec was added.

---

## Templates (in `assets/`)

Copy these into the target project and fill them in — never leave a template's example
values in place.

- `assets/spec-template.md` — feature spec with numbered ACs and an amendment block.
- `assets/component-spec-template.md` — UI feature spec with UI states, a11y, and
  responsive sections built in (frontend features).
- `assets/adr-template.md` — immutable decision record (Context / Decision / Consequences).
- `assets/conventions-template.md` — coding conventions, stack-neutral with fill-ins.
- `assets/glossary-template.md` — shared vocabulary for specs, ADRs, and code.
- `assets/openapi-stub.yaml` — minimal, lintable OpenAPI starting point (HTTP projects).
