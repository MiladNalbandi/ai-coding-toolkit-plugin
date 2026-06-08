# Workflow 6: Spec-Driven, Test-After (SDD rigor, no TDD)

> **All of spec-driven-development's rigor — spec, contract, numbered ACs,
> validation/security, review, ADR, Definition of Done — with exactly one change:
> tests are written *after* the implementation and must *pass*, not red-first.**

Use this when your team (or the task) wants the full SDD discipline but does **not** practice
test-first development. It is heavier than PBV (which drops the spec, contract, ACs, and
Definition of Done) and as rigorous as Full SDD everywhere except test timing.

This workflow does **not** duplicate the SDD step content — it reorders it. Run each step
from the `spec-driven-development` skill in the sequence below.

## When to pick this vs the neighbours

| Pick this (Workflow 6) when… | Pick instead… |
|------------------------------|----------------|
| You want spec + contract + ACs + DoD, but write tests after building | Test-first is fine → **Full SDD** |
| Touches data / auth / an API contract | No contract or data impact, trivial → **Quick build** (Workflow 1) |
| You want traceable rigor, not just a plan | A plan + gates is enough, no frozen spec → **PBV** (Workflow 5) |
| High-risk/regulatory and you want strictest mode | → **Full SDD + TDD** |

## The reordered sequence

Spec and the contract still come **first** (they define intent before code). Validation and
security are **decisions**, not tests, so they're still pinned down before coding. Only the
**test-writing step moves to after implementation**.

1. **Spec + approval gate** — per SDD Step 1 / 1a. Numbered, observable, testable ACs. Frozen once referenced.
2. **Contract** (HTTP features only) — per SDD Step 2. Update `openapi.yaml` before the code.
3. **Validation & Security rules** — per SDD Step 4. Define required/optional fields, types, auth, ownership, data-exposure rules *before* coding (these are design decisions, not tests).
4. **Implement** — per SDD Step 5 + the layer order in [`coding-structure.md`](coding-structure.md). YAGNI; thin entry points.
5. **Write tests (AFTER) — the one reordered step** — per SDD Step 3 sub-rules 3a–3k and [`testing-structure.md`](testing-structure.md). Same pyramid (≈70/25/5), AAA, hermetic isolation, factories, mocking discipline, coverage budgets. **The only difference from Full SDD: tests are written now, against the finished code, and must be _green_ — not red-first.** Every AC still gets at least one test, named after the AC, at the lowest layer that can express it. **Unit tests are mandatory**; after the suite is green, run a coverage report and confirm it meets the budget (≥80% overall, 100% on critical paths).
6. **Refactor** — per SDD Step 6. Formatter → linter → type-checker → full suite → smoke, while green.
7. **Review & Smoke** — per SDD Step 7. Human + adversarial AI review; generated smoke script.
8. **ADR** (when warranted) — per SDD Step 8.

## What changes vs Full SDD

| | Full SDD | This workflow (test-after) |
|---|---|---|
| Spec, contract, ACs, validation/security, review, ADR, DoD | ✅ | ✅ (unchanged) |
| Test timing | **Before** implement (red-first) | **After** implement |
| Test must… | **fail** first, then pass | **pass** (written against finished code) |
| Everything in Step 3 (pyramid, AAA, isolation, factories, coverage) | ✅ | ✅ (same — only the timing moves) |

## The risk you take on, and how to mitigate it

Red-first gives you a free signal: *if you can't write a clear failing test, the spec is too
vague.* Writing tests after the code loses that signal and invites **tests that merely
describe what the code happens to do** (confirmation bias).

Mitigate:
- **Review ACs against the spec before implementing** (step 3.5 mental check) — don't let the
  implementation define the ACs.
- Write each test from the **AC text**, not by reading the implementation.
- Run an **adversarial AI review** of the tests (per SDD Step 7) specifically for vague
  assertions, missing edge cases, and tests that can't fail.
- Keep the coverage budget as a floor (≥80% overall, 100% critical paths).

## Definition of Done (one bullet differs from Full SDD)

Identical to the SDD Definition of Done, except this bullet:

- Full SDD: *“Tests exist, map back to ACs, and **failed before** implementation.”*
- **This workflow:** *“Tests exist, map back to ACs, and **pass** — written immediately after implementation (not red-first). Every AC has at least one test; coverage budgets still hold.”*

Everything else (spec frozen, contract matches behavior, validation/authz explicit and
tested, sensitive data not exposed, full suite + smoke pass, PR reviewed, ADR for non-obvious
decisions, no out-of-spec behavior) is unchanged.
