# Workflow 5: Plan → Build → Verify (PBV)

> **The general-purpose disciplined flow.** Not as strict as full SDD, lighter than
> spec-driven, but heavier than quick build. Use when the change deserves a plan and
> a review gate but does not need a frozen spec or contract.
>
> Pattern: **plan → human approves plan → break into small tasks → multi-agent build →
> tests → smoke + e2e → lint → human review → done.**

## Step 1 — Write a plan and get human approval

Produce a short plan (5–15 numbered steps). Each step must be:
- Independently completable
- Small enough to assign to a single agent
- Tied to an observable outcome

Then ask the user this question and wait for their reply:

> Plan written ({{N}} steps). Approve before breaking into tasks?

Options:
1. **Approve — break into tasks and build** — Lock the plan, proceed to create the task list.
2. **Edit a step — change scope or order** — I'll ask which step number, apply changes, then re-ask.
3. **Add missing step** — I'll ask what step to add, slot it in, then re-ask.
4. **AI review the plan** — Run an adversarial AI code review of the plan; it catches missing steps, hidden scope, and ordering issues.
5. **Reject — too vague** — Hand off to clarify-loop with the original task.

Do not proceed until the user picks **Approve**.

---

## Step 2 — Break the plan into small tasks (todo tool)

Track progress with the todo tool (todowrite/todoread). For every approved plan step, add
a todo entry with:
- A short **subject** in imperative form
- The **in-progress form** for the active state
- A short note tying the task back to the plan step number

```
Add one todo per approved plan step (todowrite).
Mark a todo in_progress when starting it, completed when done (todowrite).
Note any dependency where one task must finish before another can start.
```

This gives the user live progress visibility and prevents silent reordering.

---

## Step 3 — Sequential or parallel agents?

Ask the user this question and wait for their reply:

> How should I execute the tasks?

Options:
1. **Parallel agents (Recommended)** — Spawn agents via the `multi-agent` skill — one file per agent. Fastest.
2. **Sequential (one at a time)** — Safer when tasks share files. Slower but easier to debug.

Either mode follows the **coding structure** in [`coding-structure.md`](coding-structure.md)
— layer order (schema → … → route), thin entry points, YAGNI.

### Parallel mode

Use the **multi-agent skill** to fan out:
- Conflict check first: list files each agent will write. Serialize any pair that overlaps.
- If an MCP swarm server is configured, use it to spawn the agents; otherwise use opencode's
  `task` tool to spawn one subagent per file (spawn the independent ones together so they run concurrently).
- Each agent owns exactly one file and one todo entry
- After all agents return, merge outputs and wire imports

### Sequential mode

Run tasks in dependency (layer) order. After each task: run only the relevant tests, then move on.

---

## Step 3.5 — Human review gate after implementation

Show the user the implementation diff (all files written by the agents). Then ask the user
this question and wait for their reply:

> Implementation complete. Review the code before I write tests?

Options:
1. **Approve — proceed to write tests** — Implementation matches the plan. Move on.
2. **Request changes** — I'll ask which files/lines, apply changes, then re-show the diff.
3. **AI review first** — Run an adversarial AI code review before tests are written. Each finding gets Apply / Defer / Reject.
4. **Reject — restart from Step 3 (rebuild)** — Implementation diverges from the plan. Spawn agents again with corrected prompts.

Do not proceed to Step 4 until **Approve**.

---

## Step 4 — Write tests for what was built

**Follow the testing structure** in [`testing-structure.md`](testing-structure.md) — the
pyramid, AAA, hermetic isolation, factories, mocking discipline, coverage budgets. Cover:
- **Unit** — pure logic (validators, mappers, serializers, calculators)
- **Integration** — multi-layer wiring with a real DB, transaction rolled back per test
- **Contract** — if the change touches an API or interface
- **e2e** — one happy path + one failure path (overlaps with Step 5 smoke)

Each requirement gets a test at the **lowest layer that can express it**, and **unit tests
are mandatory** for the logic that was built. Tests must be **green** before moving to smoke.
No skipped tests committed without a tracked follow-up. Then **run a coverage report** and
confirm it meets the budget in [`testing-structure.md`](testing-structure.md) (≥80% overall,
100% on critical paths).

---

## Step 4.5 — Human review gate after tests

Show the user the test diff (new test files + any test modifications) and the test run
output. Then ask the user this question and wait for their reply:

> Tests written and passing. Review them before smoke/e2e?

Options:
1. **Approve — proceed to smoke and e2e** — Tests cover the ACs at the right layer with good isolation.
2. **Request changes — missing cases or weak assertions** — I'll ask which tests/cases, add or strengthen them, then re-run.
3. **AI review the tests** — Run an adversarial AI code review; it catches vague assertions, missing edge cases, and mocking smell.
4. **Reject — tests don't match the plan/ACs** — Restart Step 4 with corrected scope.

Do not proceed to Step 5 until **Approve**.

---

## Step 5 — Smoke + e2e tests

Generate a smoke script following the **macOS BSD portability rules** (no `head -n -1`, use `curl -o + -w`, use `mktemp`, use `grep -E`):

```bash
scripts/smoke/<feature-slug>.sh
```

Cover:
- App boots, port listens
- Routes registered (`curl --head`)
- DB / queue / cache reachable
- Auth works (login / token exchange)
- Happy path returns 2xx with expected shape
- One failure path returns expected 4xx
- No sensitive fields leaked

Run the script. If any check fails → drop into the **debugging workflow** (Reproduce → Read error → Isolate layer → Hypothesize → Fix) until green.

Add a regression test in the unit/integration suite for **every** smoke failure before continuing.

---

## Step 6 — Lint, format, type-check

Ask the user this question and wait for their reply (multiple selections allowed) to pick
which checks to run (detected from the project):

> Which checks should run before review?

Options:
1. **Formatter (prettier / black / gofmt / rustfmt / pint)** — Auto-fix style. Safest first step.
2. **Linter (eslint / ruff / phpstan / golangci-lint / clippy)** — Style + correctness rules.
3. **Type checker (tsc / mypy / phpstan / cargo check)** — Verify types.
4. **Full test suite** — Run every test one more time before review.
5. **Coverage report (meets budget)** — Run coverage; confirm ≥80% overall and 100% on critical paths per testing-structure.md.
6. **Architecture rules (deptrac / archunit / dependency-cruiser)** — Layering rules — handlers don't import DB driver, etc.

Run them in order. Stop on first failure and fix inline. Do not proceed to review with a red gate.

---

## Step 7 — Verification before completion

Force evidence before any "done" claim — paste the **real output**, not "tests pass":
- Test output (paste actual output, not "tests pass")
- Lint output
- Smoke output
- Diff summary

This catches the classic "I think it works" trap.

---

## Step 8 — Human review gate

Show the user the diff, the test results, the smoke output, and the lint pass. Then ask the
user this question and wait for their reply:

> All checks pass. Review the diff above and approve?

Options:
1. **Approve — ready to commit / PR** — Diff matches the plan and tests cover the changes.
2. **Request changes** — I'll ask which files/lines and apply changes.
3. **AI review first** — Run an adversarial AI code review for a second opinion. Each finding gets an Apply / Defer / Reject decision.
4. **Reject — restart from Step 4 (tests)** — Implementation diverges from the plan.

Only when the user picks **Approve**: create the commit/PR for the change, or stop here per the user's preference.

---

## When to pick Workflow 5 vs others

| Pick PBV (Workflow 5) when... | Pick something else when... |
|-------------------------------|------------------------------|
| The change deserves a plan but not a frozen spec | Public API or data contract → full SDD |
| Multiple files, multiple layers | Single one-line fix → quick build |
| You want multi-agent + lint + verification gates | The idea is still fuzzy → brainstorming |
| You want clear human approval checkpoints | This is a bug → systematic debugging loop |
