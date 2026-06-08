# SDD Workflow — Full Detail

This is the long-form version of the loop summarized in `SKILL.md`. It is written to be
copied into a project as `docs/sdd/00-workflow.md` and lightly edited so its examples use
the project's real stack. The loop is fixed across languages; only the tool names change.

```
1. Spec → 2. Contract → 3. Tests → 4. Validation+Security
   → 5. Implement → 6. Refactor → 7. Review+Smoke → 8. ADR
   ↑________ re-spec when a decision invalidates the original intent ________|
```

> The system should say what it means, test what it promises, and implement only what was
> agreed.

---

## 1. Spec

Specs live in `docs/specs/NNN-feature-name.md` (zero-padded sequence). A spec is the
human-readable contract between a person and the codebase, written **before**
implementation.

### Required sections

| Section             | Purpose                                                                   |
|---------------------|---------------------------------------------------------------------------|
| Goal                | One sentence describing the problem being solved.                         |
| User stories        | "As a X, I want Y, so that Z."                                            |
| Acceptance criteria | Numbered, testable, observable outcomes (AC-001, AC-002, …).             |
| Endpoints           | Method, path, request/response body, status codes. HTTP-facing only.      |
| Validation rules    | Required fields, formats, lengths, ranges, allowed values.                |
| Authorization rules | Who can do this, and who cannot.                                          |
| Domain rules        | Business invariants ("a draft has no published_at").                      |
| Edge cases          | Failure modes the implementation must handle.                             |
| Security concerns   | Abuse cases, data exposure, ownership, sensitive fields.                  |
| Out of scope        | Things explicitly excluded this iteration.                                |

### Acceptance criteria

Numbered so tests can reference them. Each must be observable from outside the system,
testable, specific, and small enough to map to one or more tests.

```
AC-001 A user can create a draft without a published_at value.
AC-002 A published item must have a published_at value.
AC-003 Creating an item without a title returns a 422 validation error.
AC-004 Guests cannot create items.
AC-005 A user cannot update another user's item.
AC-006 Unauthorized access returns 403 and reveals no private data.
AC-007 Responses do not expose internal-only fields.
```

### Freezing a spec

A spec is **frozen** once any test references it. After that, do not silently rewrite it.
Change it only via:

1. A dated amendment block at the bottom of the same spec, or
2. A new follow-up spec that supersedes the original.

```md
## Amendment — 2026-06-02
AC-004 changed to allow admins to create items on behalf of users.
Reason: moderation workflow requires admin-created items.
```

The goal is traceability, not bureaucracy.

---

## 2. Contract (HTTP-facing features)

The machine-readable counterpart of the spec. For REST, `docs/api/openapi.yaml`; for
gRPC, the `.proto`; for GraphQL, the schema. Update it **before** application code. Prose
spec and contract must agree.

Rules:
- Versioned path prefix (e.g. `/api/v1/...`).
- Every endpoint has ≥1 success and ≥1 error response.
- Request and response schemas are explicit; shared schemas are reused, not duplicated.
- Contract validation matches application validation; error shapes match real responses.
- Lint in CI (e.g. `npx @redocly/cli lint docs/api/openapi.yaml`, `buf lint`, etc.).

Non-HTTP changes skip this step but the spec must state:
`This change does not affect the API contract.`

---

## 3. Failing Tests

Written against the spec and contract, **before** the implementation exists. They must
fail first. If you cannot write a clear test, the spec is too vague — go back to step 1.

### Test layers (map to your stack's directories)

| Layer        | Purpose                                                        |
|--------------|----------------------------------------------------------------|
| Feature/API  | Exercise endpoints from outside: request in, response out.     |
| Feature/Web  | Exercise rendered pages, redirects, form submissions.          |
| Unit         | Pure logic in use-cases, services, domain objects.             |
| Architecture | Enforce layering rules and dependency boundaries.              |
| Smoke        | App boots and the main path works end-to-end.                  |

Name tests after acceptance criteria where practical, e.g.
`ac_003_creating_an_item_without_a_title_returns_422`. Security-sensitive behavior
(guests blocked, ownership enforced, internal fields hidden) **must** be covered.

---

## 4. Validation and Security

Not cleanup — part of the feature contract. Decide how input is accepted, rejected,
authorized, and safely exposed before implementing.

### Validation checklist
Required fields · optional fields · types · format (email/UUID/date/URL/slug) · length ·
range · allowed values (enum) · cross-field rules · uniqueness · failure response shape.

Validation rules must be represented in all three places: the input-validation layer, the
contract, and the feature tests.

### Security checklist
Authentication (logged-in / token / guest) · authorization (who may act) · ownership
(own records only) · roles & permissions · data exposure (private/internal fields) · mass
assignment (fields the user must not control) · rate limiting · error handling (no stack
traces, SQL, or private data leaked) · auditability · abuse cases.

Security-sensitive behavior must have tests.

### Placement rules
Do not rely on the entry-point handler to "remember" rules. Use the right layer:

| Concern             | Preferred location                       |
|---------------------|------------------------------------------|
| Authentication      | Middleware                               |
| Authorization       | Policy / guard / permission unit         |
| Input validation    | Dedicated validator (form request, model)|
| Business invariants | Use-case / service / domain object       |
| Output filtering    | Serializer / resource / DTO              |
| Rate limiting       | Middleware or route configuration        |
| Audit logging       | Dedicated logging / audit service        |

Entry points coordinate work. They do not hold business rules, authorization, or complex
validation.

---

## 5. Implementation

Write the minimum code to make the tests pass. Default order (rename to your stack's
artefacts):

```
schema/migration → entity/model → test factory/fixture → authorization rule
→ use-case/action → input validator → output serializer → handler/endpoint → route
```

Stop when green. **YAGNI is load-bearing** — add no field, endpoint, abstraction, helper,
permission, or behavior the spec did not ask for.

Rules:
- Keep entry points thin.
- Keep validation, authorization, and data access out of entry points where possible.
- Prefer use-cases/services for application behavior, serializers for output shape,
  policies for ownership/permission checks.
- Do not expose data-layer internals directly in responses.
- Do not add "future-proof" code unless the spec requires it.

---

## 6. Refactor

Once green, clean up without changing behavior. Run the toolchain (substitute your stack's
tools — see `references/stack-mapping.md`):

```
<formatter> → <automated refactorer, if any> → <static analyzer> → <full test suite>
```

Refactor freely: remove duplication, improve names, simplify control flow, extract
helpers, clarify boundaries, delete dead code. The public contract must not change unless
the spec and contract were updated first. Refactor only while the suite stays green.

---

## 7. Review and Smoke Test

Green is not mergeable. Before merge:

### Review
A human confirms: implementation matches spec; contract matches real behavior; ACs are
covered by tests; validation/authorization/ownership are explicit and tested; sensitive
data is not exposed; error responses are intentional; no out-of-spec behavior was added;
the code follows the intended architecture; entry points are thin; non-obvious decisions
have an ADR; CI passes. Review protects the contract, not just style.

### Smoke test
Small, fast, boring. Prove: the app boots; routes are registered; **the database is
reachable**; auth works (if relevant); the changed path works once happy and once failing;
no unexpected 500; the response shape matches the contract.

If smoke fails, do not patch around it — return to the earliest broken step
(spec → contract → tests → validation/security → implementation).

---

## 8. ADR

Architecture Decision Records live in `docs/adr/NNNN-title.md`. Write one when a decision
is non-obvious or likely to matter later: choosing a dependency; choosing a column type;
introducing an architectural pattern; rejecting a likely option; a
simplicity/performance/flexibility tradeoff; a security-model or authorization change; an
API-versioning change; choosing queue/cache/search/storage infra.

ADRs are **immutable** once merged. A later decision creates a new ADR that supersedes the
earlier one by number. Use `assets/adr-template.md`.

---

## Definition of Done

See the matching list in `SKILL.md`. A change merges only when spec, contract, tests,
validation, security, toolchain, smoke, review, and (if needed) ADR are all satisfied, and
nothing outside the spec was added.

---

## Worked example (HTTP, database-backed)

```
1. docs/specs/002-items.md            human-readable intent, numbered ACs
2. docs/api/openapi.yaml              add POST /api/v1/items
3. tests/.../CreateItemTest           red tests against spec + contract
4. validation/security                title required; guests blocked; published_at not
                                      user-settable; internal fields hidden
5. implementation                     migration → model → factory → policy → use-case
                                      → validator → serializer → handler → route
6. toolchain                          <format> → <static analysis> → <test --parallel>
7. review + smoke                     route list; smoke filter; one happy curl/request
8. ADR (if warranted)                 docs/adr/0003-search-engine-choice.md
```

That sequence is what merges. If the sequence breaks, the PR explains why.
