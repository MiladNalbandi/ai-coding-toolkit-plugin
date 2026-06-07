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

A process for shipping every meaningful change — from a one-line fix to a new
feature — through the same loop, so that **intent, contract, validation, security,
tests, and code never drift apart**. A year later, anyone reading the repo can answer
*why* a line exists, not just *what* it does.

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
Run the project's toolchain (formatter → automated refactorer → static analyzer → full
test suite). Remove duplication, improve names, simplify flow, clarify boundaries, delete
dead code. The public contract must not change here unless the spec and contract were
updated first. Refactor only while the suite stays green.

### 7. Review & Smoke — *green ≠ mergeable*
Human review confirms the implementation matches the spec, the contract matches real
behavior, ACs are covered, validation/authorization/ownership are explicit and tested,
sensitive data is not exposed, and no out-of-spec behavior crept in. Then a small, fast,
**boring** smoke test proves the app boots, routes are registered, the database is
reachable, auth works, the changed path works once happy and once failing, and the
response shape matches the contract. If smoke fails, return to the earliest broken step —
do not patch around it.

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
