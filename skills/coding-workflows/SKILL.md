---
name: coding-workflows
description: >
  Step-by-step workflows for feature development, debugging, architecture decisions,
  and code review, each with ready-to-use prompts for every stage. Use when starting
  any coding task — building a feature, fixing a bug, designing a system, or reviewing
  code. Hands off to spec-driven-development for rigorous feature work. Trigger on
  "how do I approach this", "debug this", "review my code", "design the architecture for".
command: /workflow
---

# Coding Workflows — AI-Assisted Development

> **Applying to:** $ARGUMENTS

Step-by-step processes for the most common coding tasks. Follow the order inside each
workflow. Never skip Clarify and Design — they prevent 80% of rework.

This file is an index. Each workflow lives in its own reference file under `references/`.
**Read only the one you need** — don't load all five.

---

## Pick a workflow

| Workflow | Use when | Reference |
|----------|----------|-----------|
| **1. Feature Development** | Building something new (lightweight; non-traceable) | [`references/feature-development.md`](references/feature-development.md) |
| **2. Debugging** | Something is broken — find and fix the root cause | [`references/debugging.md`](references/debugging.md) |
| **3. Architecture** | An (often irreversible) design decision between options | [`references/architecture.md`](references/architecture.md) |
| **4. Code Review** | Reviewing a diff across correctness / security / perf / maintainability | [`references/code-review.md`](references/code-review.md) |
| **5. Plan → Build → Verify (PBV)** | A multi-file change that deserves a plan, multi-agent build, and human gates | [`references/plan-build-verify.md`](references/plan-build-verify.md) |
| **6. Spec-Driven, Test-After** | You want full SDD rigor (spec, contract, ACs, DoD) but write tests *after* building, not test-first | [`references/spec-driven-test-after.md`](references/spec-driven-test-after.md) |

### Shared structures (used by the implement + test steps)

Any workflow that **writes code** or **writes tests** follows these two references so the
output is consistent across every task:

- **Coding structure** — [`references/coding-structure.md`](references/coding-structure.md): the layer order (schema → model → factory → authz → use-case → validator → serializer → handler → route), placement rules, thin entry points, YAGNI.
- **Testing structure** — [`references/testing-structure.md`](references/testing-structure.md): the pyramid (unit / integration / contract / e2e), AAA, hermetic isolation, factories, mocking discipline, coverage budgets, layout.

### Routing rules

- **Requirements still fuzzy?** → run the `clarify-loop` skill first, then come back.
- **Change has acceptance criteria worth numbering, or touches validation / authorization / an API contract?** → use `spec-driven-development` instead of Workflow 1.
- **Multi-file change with review checkpoints but no frozen spec?** → Workflow 5 (PBV).
- **Want full SDD rigor (spec/contract/ACs/DoD) but your team writes tests _after_ building, not test-first?** → Workflow 6 (Spec-Driven, Test-After).
- **Just a bug?** → Workflow 2, or `superpowers:systematic-debugging` for a deeper loop.

---

## When to pick PBV vs others

| Pick PBV (Workflow 5) when... | Pick something else when... |
|-------------------------------|------------------------------|
| The change deserves a plan but not a frozen spec | Public API or data contract → full SDD |
| Multiple files, multiple layers | Single one-line fix → quick build |
| You want multi-agent + lint + verification gates | The idea is still fuzzy → brainstorming |
| You want clear human approval checkpoints | This is a bug → systematic-debugging |

---

## Key Principles

| Rule | Why |
|------|-----|
| Always clarify before coding | Removes ambiguity. Prevents building the wrong thing. |
| Always design before implementing | Gets plan approval. 10 minutes of design saves 2 hours of rework. |
| Implement one chunk at a time | Keeps context small. Makes reviewing easier. |
| Write tests before marking done | No test = no confidence. Tests also document behavior. |
| Every fix needs a regression test | A bug that came back once will come back again without a test. |
| Document architecture decisions | Future-you will not remember why. Your teammates will be confused. |

---

## Related skills

- `clarify-loop` — pin down scope and numbered ACs before any of these workflows.
- `spec-driven-development` — the rigorous spec → contract → red-tests → implement loop.
- `multi-agent` — fan-out execution used by Workflow 1 (parallel) and Workflow 5.
- `superpowers:systematic-debugging`, `superpowers:verification-before-completion`,
  `superpowers:requesting-code-review`, `superpowers:finishing-a-development-branch`.
