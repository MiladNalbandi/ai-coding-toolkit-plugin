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

Then **use `AskUserQuestion` (single-select)** to approve:

```json
{
  "question": "Plan written ({{N}} steps). Approve before breaking into tasks?",
  "header": "Plan approval",
  "multiSelect": false,
  "options": [
    { "label": "✅ Approve — break into tasks and build", "description": "Lock the plan, proceed to TaskCreate." },
    { "label": "✏️  Edit a step — change scope or order", "description": "I'll ask which step number, apply changes, then re-ask." },
    { "label": "➕ Add missing step", "description": "I'll ask what step to add, slot it in, then re-ask." },
    { "label": "🤖 AI review the plan (superpowers:requesting-code-review)", "description": "Adversarial review catches missing steps, hidden scope, ordering issues." },
    { "label": "❌ Reject — too vague", "description": "Hand off to clarify-loop with the original task." }
  ]
}
```

Do not proceed until the user picks **Approve**.

---

## Step 2 — Break the plan into small tasks (TaskCreate)

For every approved plan step, call `TaskCreate` with:
- A short **subject** in imperative form
- The **activeForm** for the spinner
- A short description tying the task back to the plan step number

```
TaskCreate per plan step
TaskUpdate to set in_progress when starting, completed when done
Use addBlockedBy when one task depends on another
```

This gives the user live progress visibility and prevents silent reordering.

---

## Step 3 — Sequential or parallel agents?

**Use `AskUserQuestion` (single-select)**:

```json
{
  "question": "How should I execute the tasks?",
  "header": "Execution mode",
  "multiSelect": false,
  "options": [
    { "label": "Parallel agents (Recommended)", "description": "Spawn agents via ai-coding-toolkit:multi-agent — one file per agent. Fastest." },
    { "label": "Sequential (one at a time)", "description": "Safer when tasks share files. Slower but easier to debug." }
  ]
}
```

### Parallel mode

Use the **multi-agent skill** to fan out:
- Conflict check first: list files each agent will write. Serialize any pair that overlaps.
- Spawn via `ruflo agent_spawn` if Ruflo MCP is available; else use parallel `Agent` tool calls in one message
- Each agent owns exactly one file and one TaskList task
- After all agents return, merge outputs and wire imports

### Sequential mode

Run tasks in dependency order. After each task: run only the relevant tests, then move on.

---

## Step 3.5 — Human review gate after implementation

Show the user the implementation diff (all files written by the agents). Then **use `AskUserQuestion` (single-select)**:

```json
{
  "question": "Implementation complete. Review the code before I write tests?",
  "header": "Post-impl review",
  "multiSelect": false,
  "options": [
    { "label": "✅ Approve — proceed to write tests", "description": "Implementation matches the plan. Move on." },
    { "label": "✏️  Request changes", "description": "I'll ask which files/lines, apply changes, then re-show the diff." },
    { "label": "🤖 AI review first (superpowers:requesting-code-review)", "description": "Adversarial review before tests are written. Each finding gets Apply / Defer / Reject." },
    { "label": "❌ Reject — restart from Step 3 (rebuild)", "description": "Implementation diverges from the plan. Spawn agents again with corrected prompts." }
  ]
}
```

Do not proceed to Step 4 until **Approve**.

---

## Step 4 — Write tests for what was built

Per Step 3 of SDD (test pyramid, AAA, isolation, factories, mocking). Cover:
- **Unit** — pure logic (validators, mappers, calculators)
- **Integration** — multi-layer wiring with real DB rollback per test
- **Contract** — if the change touches an API or interface

Tests must be **green** before moving to smoke. No skipped tests committed without a tracked follow-up.

---

## Step 4.5 — Human review gate after tests

Show the user the test diff (new test files + any test modifications) and the test run output. Then **use `AskUserQuestion` (single-select)**:

```json
{
  "question": "Tests written and passing. Review them before smoke/e2e?",
  "header": "Post-test review",
  "multiSelect": false,
  "options": [
    { "label": "✅ Approve — proceed to smoke and e2e", "description": "Tests cover the ACs at the right layer with good isolation." },
    { "label": "✏️  Request changes — missing cases or weak assertions", "description": "I'll ask which tests/cases, add or strengthen them, then re-run." },
    { "label": "🤖 AI review the tests (superpowers:requesting-code-review)", "description": "Adversarial review catches vague assertions, missing edge cases, mocking smell." },
    { "label": "❌ Reject — tests don't match the plan/ACs", "description": "Restart Step 4 with corrected scope." }
  ]
}
```

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

**Use `AskUserQuestion` (multiSelect: true)** to pick which checks to run (detected from the project):

```json
{
  "question": "Which checks should run before review?",
  "header": "Quality gate",
  "multiSelect": true,
  "options": [
    { "label": "Formatter (prettier / black / gofmt / rustfmt / pint)", "description": "Auto-fix style. Safest first step." },
    { "label": "Linter (eslint / ruff / phpstan / golangci-lint / clippy)", "description": "Style + correctness rules." },
    { "label": "Type checker (tsc / mypy / phpstan / cargo check)", "description": "Verify types." },
    { "label": "Full test suite", "description": "Run every test one more time before review." },
    { "label": "Architecture rules (deptrac / archunit / dependency-cruiser)", "description": "Layering rules — handlers don't import DB driver, etc." }
  ]
}
```

Run them in order. Stop on first failure and fix inline. Do not proceed to review with a red gate.

---

## Step 7 — Verification before completion

Invoke `superpowers:verification-before-completion` — it **forces evidence** before any "done" claim:
- Test output (paste actual output, not "tests pass")
- Lint output
- Smoke output
- Diff summary

This catches the classic "I think it works" trap.

---

## Step 8 — Human review gate

Show the user the diff, the test results, the smoke output, and the lint pass. Then **use `AskUserQuestion` (single-select)**:

```json
{
  "question": "All checks pass. Review the diff above and approve?",
  "header": "Final review",
  "multiSelect": false,
  "options": [
    { "label": "✅ Approve — ready to commit / PR", "description": "Diff matches the plan and tests cover the changes." },
    { "label": "✏️  Request changes", "description": "I'll ask which files/lines and apply changes." },
    { "label": "🤖 AI review (superpowers:requesting-code-review) first", "description": "Adversarial second opinion. Each finding gets an Apply / Defer / Reject decision." },
    { "label": "❌ Reject — restart from Step 4 (tests)", "description": "Implementation diverges from the plan." }
  ]
}
```

Only when the user picks **Approve**: hand off to `superpowers:finishing-a-development-branch` for the commit/PR step, or stop here per the user's preference.

---

## When to pick Workflow 5 vs others

| Pick PBV (Workflow 5) when... | Pick something else when... |
|-------------------------------|------------------------------|
| The change deserves a plan but not a frozen spec | Public API or data contract → full SDD |
| Multiple files, multiple layers | Single one-line fix → quick build |
| You want multi-agent + lint + verification gates | The idea is still fuzzy → brainstorming |
| You want clear human approval checkpoints | This is a bug → systematic-debugging |
