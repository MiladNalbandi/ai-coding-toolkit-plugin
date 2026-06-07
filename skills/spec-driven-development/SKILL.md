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
---

# Spec-Driven Development (SDD)

> **Task:** $ARGUMENTS

---

## Step −1 — Which flow do you want to run?

Before SDD-specific questions, ask the user **which overall flow** fits the task.
This skill is most powerful when chained with sibling skills.

```
What flow do you want for `$ARGUMENTS`? (default: 4)

  1. Clarify only         — just produce numbered ACs (clarify-loop)
                            Use when: you'll implement later or pass to someone else
  2. Brainstorm → design  — superpowers:brainstorming → writing a design doc
                            Use when: the idea is fuzzy and needs shape first
  3. Quick build          — coding-workflows lightweight flow (no spec/contract)
                            Use when: trivial change, no public contract or data impact
  4. Full SDD             — spec → contract → red tests → implement → smoke
                            Use when: data, auth, or API contract is touched
  5. Full SDD + TDD       — full SDD + superpowers:test-driven-development discipline
                            Use when: high-risk or regulatory work — strictest mode
  6. Debug existing bug   — superpowers:systematic-debugging
                            Use when: this is a bug, not a feature
  7. Architecture only    — coding-workflows architecture flow + ADR
                            Use when: you need a decision, not implementation

Reply with the number, or "auto" to let me infer from the task.
```

Store as `<flow>`.

### Routing table

| Choice | First skill | Then |
|--------|-------------|------|
| 1 | `clarify-loop` | stop |
| 2 | `superpowers:brainstorming` | `superpowers:writing-plans` |
| 3 | `coding-workflows` | `superpowers:verification-before-completion` |
| 4 | `clarify-loop` (ACs) | continue with this SDD skill (Step 0 below) |
| 5 | `clarify-loop` (ACs) | this SDD skill + `superpowers:test-driven-development` for tests |
| 6 | `superpowers:systematic-debugging` | regression test in this codebase |
| 7 | `coding-workflows` (Workflow 3) | write ADR via this skill (Step 8) |

If the user picks anything other than **4** or **5**, hand off and exit this skill.

For **4** and **5**, continue with Step 0 below.

---

A process for shipping every meaningful change — from a one-line fix to a new
feature — through the same loop, so that **intent, contract, validation, security,
tests, and code never drift apart**. A year later, anyone reading the repo can answer
*why* a line exists, not just *what* it does.

---

## Step 0 — Ask the user which steps to run

Before running any step, ask the user **two questions** and wait for answers.

### Question 1 — Which steps do you want to include?

Present a checklist. Default = all checked. User can deselect by step number.

```
Pick which SDD steps to run for this task (default: all):

  [x] 1. Spec — write docs/specs/NNN-feature.md with numbered ACs
  [x] 2. Contract — update docs/api/openapi.yaml (HTTP features only)
  [x] 3. Failing tests — red-first tests mapped to ACs
  [x] 4. Validation + Security checklist
  [x] 5. Implement — minimum code to make tests green
  [x] 6. Refactor — formatter, linter, static analyzer, full test suite
  [x] 7. Review + Smoke test — final correctness pass
  [x] 8. ADR — record non-obvious decisions (skip if no decision worth recording)

Reply with step numbers to SKIP (e.g. "skip 2, 8") or "all" to keep everything.
```

**Sensible skip combinations:**
- Non-HTTP feature: skip 2 (Contract)
- Trivial bug fix: skip 2, 8 (Contract + ADR)
- Documentation-only change: skip 2, 3, 5 (Contract, Tests, Implement)
- Bootstrapping SDD in a fresh repo: run all + scaffold templates first

Store selected steps as `<active-steps>`. Skip any step not in this set.

### Question 2 — Which lint/test tools should the Refactor step run?

Detect what's available in the project, then present a checklist. Default = all detected, all checked.

```
For Step 6 (Refactor), which tools should I run? (detected from your project)

  [x] Formatter:     {{e.g. pint / prettier / black / gofmt / rustfmt}}
  [x] Auto-refactor: {{e.g. rector / ts-morph / autoflake — or "skip if N/A"}}
  [x] Linter:        {{e.g. phpstan / eslint / ruff / golangci-lint / clippy}}
  [x] Type checker:  {{e.g. mypy / tsc --noEmit / phpstan / cargo check}}
  [x] Full test:     {{e.g. composer test / pytest / npm test / go test ./... / cargo test}}
  [x] Smoke test:    {{e.g. curl health endpoint, app boot check}}

Reply with tool names to SKIP, or "all" to run everything.
You can also add tools I missed (e.g. "+ phpcs", "+ vitest run").
```

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
*must* have tests. Test layers: feature/API (outside-in), unit (pure logic), architecture
(layering rules), smoke (boots + happy path).

### 4. Validation & Security — *part of the contract, not cleanup*
Before implementing, answer the validation checklist (required/optional fields, types,
format, length, range, allowed values, cross-field rules, uniqueness, failure response)
and the security checklist (authn, authz, ownership, roles, data exposure, mass
assignment, rate limiting, error leakage, auditability, abuse cases). Put each concern in
its correct layer (see placement table in `references/workflow.md`) — never rely on the
entry-point handler to "remember" rules.

### 5. Implement — *minimum code to make tests green*
Default order: schema/migration → entity/model → test factory/fixture → authorization
rule → use-case/action → input validator → output serializer → handler/endpoint → route.
Stop when green. **YAGNI is load-bearing**: add no field, endpoint, abstraction, helper,
permission, or behavior the spec did not ask for. Keep entry points thin; push
validation, authorization, business rules, data access, and output shaping into their
dedicated layers.

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

#### 7a. Ask the user which review mode

Before running review, ask:

```
How do you want to review this change? (default: both)

  [x] Human review        — I'll pause and wait for you to read the diff
  [x] AI review (superpowers:requesting-code-review) — adversarial bot review
  [x] Smoke test          — generated script + auto-fix loop on failures

Reply with what to SKIP (e.g. "skip human") or "all".
```

Store as `<review-mode>`.

#### 7b. Human review (if selected)

Pause and ask the user to review:

> "Implementation ready. Please review the diff for: spec match, contract match, AC coverage, validation/authorization/ownership explicit and tested, no sensitive data exposed, no out-of-spec behavior. Reply 'approved' or list changes."

Wait for response. If changes requested → apply them and re-ask. Only proceed when approved.

#### 7c. AI review (if selected)

Invoke the superpowers code review skill:

```
Skill: superpowers:requesting-code-review
```

That skill spawns an adversarial reviewer that:
- Tries to find logic bugs and edge cases
- Tries to find security holes (injection, auth bypass, data leaks)
- Tries to find performance issues (N+1, blocking calls)
- Checks the diff against the spec and contract

For each finding the AI reviewer produces, decide:
- **Apply** — fix it, re-run failing tests
- **Defer** — log it as a follow-up issue with rationale
- **Reject** — explain why the reviewer is wrong (and update the spec if it caused confusion)

Loop until the AI reviewer returns no new high-severity findings.

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

5. **Final review gate after fixes** — when all smoke checks pass, ask the user again:

   ```
   All smoke checks pass after fixes. Run final review? (default: yes)

     [x] Human review of the fix diff
     [x] AI review (superpowers:requesting-code-review) on the fix
     [x] Run superpowers:verification-before-completion checklist
   ```

   The `superpowers:verification-before-completion` skill is mandatory before
   claiming "done" — it forces evidence (test output, smoke output, lint pass)
   before any success claim, commit, or PR.

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
