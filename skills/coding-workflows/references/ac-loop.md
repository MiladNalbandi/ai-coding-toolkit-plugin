# The AC Loop — one acceptance criterion at a time

> **The unit of work is the acceptance criterion, not the feature.**
> For each AC: write its test, watch it fail, write the minimum code that passes, commit
> each half separately, then stop at a review gate before touching the next AC.

This file is the single definition of the loop. `spec-driven-development` (Steps 3 + 5),
Workflow 1 (Feature Development), Workflow 5 (PBV), and Workflow 6 (Spec-Driven,
Test-After) all run it — they differ only in the **variant** they pick (see §6) and in what
they do before and after. Do not restate the loop elsewhere; link here.

## Why

Writing every failing test, implementing everything, then reviewing once produces one
enormous diff reviewed at the most expensive possible moment. Small committed steps mean:
a reviewer sees one behavior at a time, a bisect lands on one AC, and a wrong direction
costs one criterion instead of a feature.

---

## 1. The loop

```
                ACs from clarify-loop / the spec
                          │
                          ▼
    ┌──────────── for each AC-NNN, in order ────────────┐
    │                                                    │
    │   ┌─ RED ────────────────────────────────────┐     │
    │   │ write the test(s) for AC-NNN only        │     │
    │   │ run → MUST FAIL for the right reason     │     │
    │   │ commit  test(AC-NNN): <ac text>          │     │
    │   └──────────────────┬───────────────────────┘     │
    │                      ▼                             │
    │   ┌─ GREEN ──────────────────────────────────┐     │
    │   │ minimum code to pass — YAGNI             │     │
    │   │ run → this AC green, no other AC broken  │     │
    │   │ commit  feat(AC-NNN): <ac text>          │     │
    │   └──────────────────┬───────────────────────┘     │
    │                      ▼                             │
    │   ┌─ GATE (AskUserQuestion) ─────────────────┐     │
    │   │ 7 options — §4                           │     │
    │   └──────────────────┬───────────────────────┘     │
    │          ┌───────────┼───────────┐                 │
    │          ▼           ▼           ▼                 │
    │      approve     refactor    back to RED           │
    │          │      (3rd commit)      │                │
    │          │           │            └──── retry ─────┤
    └──────────┴───────────┴──────────────── next AC ────┘
                          │
                          ▼
          all ACs green → whole-feature refactor
                        → review + smoke → ADR
```

**Order the ACs before starting.** Run them in dependency order (an AC whose test needs a
table, a model, or an endpoint that another AC creates comes second), not in numeric order.
State the chosen order once, up front, and keep it visible in the progress board.

---

## 2. The phases

### RED — write the failing test

- Write **only** the test(s) for this AC. No test for a later AC, no scaffolding "while
  you're in there".
- Name the test after the AC: `ac_003_creating_a_post_without_a_title_returns_422`.
- Run it. It must fail **for the right reason** — the assertion, not an import error, a
  typo, or a missing fixture. A test that errors before reaching the assertion has not
  gone red; fix the test and re-run.
- Pick the layer from `testing-structure.md`: the lowest layer that can express the AC.
- Commit (see §3).

**If you cannot write a clear failing test, the AC is too vague.** Stop and take it back to
the spec — that signal is the whole point of going red first.

### GREEN — the minimum code that passes

- Implement in the layer order from `coding-structure.md`, but only as far as this AC
  needs: schema → model → factory → authz → use-case → validator → serializer → handler
  → route.
- **YAGNI is load-bearing.** No field, endpoint, abstraction, helper, permission, or
  branch the AC did not ask for. If you find yourself writing code no current test drives,
  it belongs to a later AC — stop.
- Run **this AC's test** (must pass) and then the **full suite** (no other AC may break).
  A regression here is part of this AC's work, not a follow-up.
- Commit (see §3).

### GATE — stop and decide

Run the gate in §4. Do not start the next AC before it returns approve.

### REFACTOR — optional, only while green

Chosen from the gate. Behavior must not change, the suite must stay green, and it gets its
own commit. Big structural cleanups that span several ACs belong to the whole-feature
refactor step after the loop, not here.

---

## 3. Commits

| Phase | Message | Contains |
|-------|---------|----------|
| RED | `test(AC-003): reject a post with no title` | test files only |
| GREEN | `feat(AC-003): reject a post with no title` | implementation only |
| REFACTOR | `refactor(AC-003): extract TitleRule` | no behavior change |
| REVIEW FIX | `fix(AC-003): review — validate before authorize` | one finding per commit |

Every commit body carries a trailer so the chain reads in both directions:

```
Spec: docs/specs/007-post-bookmarks.md#AC-003
```

A red commit stays **local** until its green pair exists — the loop never pushes a
knowingly failing tree.

### The commit-style knob

Ask once, before the first AC, and store as `<commit-style>`:

```json
{
  "question": "How should I commit as I work through the ACs?",
  "header": "Commit style",
  "multiSelect": false,
  "options": [
    { "label": "Two commits per AC (Recommended)", "description": "test(AC-NNN) then feat(AC-NNN). The red→green history is visible and bisectable. Red commits are not pushed until their green pair exists." },
    { "label": "One commit per AC (squash)", "description": "Test and implementation land together as feat(AC-NNN). Pick this when CI runs on every commit and must never be red." },
    { "label": "One commit at the end", "description": "No commits during the loop; a single commit once every AC is green. Pick this if you squash-merge anyway." },
    { "label": "No commits — leave it in the working tree", "description": "I run the loop and the gates but never call git commit. You commit by hand." }
  ]
}
```

The gate, the tests, and the review all work identically under every style — only the
`git commit` calls change.

---

## 4. The gate

`AskUserQuestion`, single-select, after every AC:

```json
{
  "question": "AC-003 is green (2/9 done). Test: test(AC-003) · Impl: feat(AC-003). What next?",
  "header": "AC-003 gate",
  "multiSelect": false,
  "options": [
    { "label": "✅ Approve — next AC", "description": "AC-003 is done. Move to AC-004." },
    { "label": "👤 Human review — show me the diff", "description": "Print this AC's diff and wait. You can then approve, request changes, or reject." },
    { "label": "🤖 AI review this AC", "description": "Adversarial review of this AC's diff only. Each finding gets Apply / Defer / Reject, then back to this gate." },
    { "label": "📝 Add my recommendation, then AI review on top", "description": "You type a change; I apply it as fix(AC-003), then run the AI review against the amended diff and bring the findings back here." },
    { "label": "♻️ Refactor now (while green)", "description": "Clean up this AC's code with the suite green, commit refactor(AC-003), re-run tests, return to this gate." },
    { "label": "⏭ Approve all remaining ACs", "description": "Run the rest of the loop without gating. The whole-feature review still happens after the loop." },
    { "label": "❌ Reject — back to RED for this AC", "description": "The test or the implementation is wrong. Reset to the red phase for AC-003 with your notes." }
  ]
}
```

### Semantics

- **Approve** → next AC. **Approve all remaining** → run the rest ungated. **Reject** →
  back to RED. Every other option **returns to this same gate**, so
  *AI review → add a recommendation → AI review again → refactor → approve* is a chain the
  user can walk one step at a time. That chain is the point of the gate.
- **Human review** shows this AC's diff only (`git show` the two commits, or the unstaged
  diff under `<commit-style> = none`), never the whole feature.
- **AI review** runs an adversarial review scoped to this AC's diff. Each finding gets a
  per-finding Apply / Defer / Reject decision. Applied findings land as separate
  `fix(AC-NNN): review — …` commits, and the tests re-run before returning to the gate.
- **Add my recommendation** is a two-hop, in this order: (1) collect the user's change as
  free text, apply it, commit it as `fix(AC-NNN)`; (2) *then* run the AI review against the
  amended diff. The reviewer must see the recommendation, not the code it replaced.
- **Findings that belong to another AC** are recorded and replayed at that AC's gate. Never
  fix a later AC's finding early — it produces code no current test drives.
- **Reject** branches by what was wrong: the *test* → amend the red commit; the
  *implementation* → revert the green commit and redo it; the *AC itself* → leave the loop
  and open a dated amendment block in the spec. Never silently edit a frozen AC.

### The progress board

Print it every time the gate asks:

```
  AC-001 ✔ test+feat   AC-004 ⬜             AC-007 ⬜
  AC-002 ✔ test+feat   AC-005 ⬜             AC-008 ⬜
  AC-003 ▶ at gate     AC-006 ⬜             AC-009 ⬜
  ────────────────────────────────────────────────────
  2/9 done · 41 tests green · 0 skipped
```

Markers: `✔` done · `▶` current · `⬜` not started · `↺` rejected once, redoing ·
`⚠` deferred finding attached.

---

## 5. When something goes wrong

| Situation | Do this |
|-----------|---------|
| The new test passes before you write any code | The behavior already exists, or the test asserts nothing. Prove it can fail (break the code path once), then either delete the AC as already-met or fix the assertion. |
| The test errors instead of failing | It has not gone red. Fix the test setup first — an import error is not a red test. |
| Making this AC green breaks an earlier AC | Part of this AC. Fix it now, in the green phase, before the gate. If the two ACs genuinely contradict, stop: that is a spec conflict, not a code problem. |
| The AC needs a schema change | Leave the loop for the `database-migrations` skill, land the migration, then come back to RED. Never invent a column mid-green-phase. |
| The AC is too big to hold one test | Split it in the spec (AC-003 → AC-003a/AC-003b) via an amendment; do not silently implement half. |
| The gate keeps producing findings on the same AC | Stop looping on symptoms — the design is wrong. Take it to the whole-feature refactor or back to the spec. |

---

## 6. The three variants

| Variant | Order | Used by |
|---------|-------|---------|
| **TDD** (default) | RED → GREEN → gate | `spec-driven-development` (Full SDD, SDD + TDD) |
| **Test-after** | GREEN (`feat`) → TEST (`test`, must pass) → gate | Workflow 6 (Spec-Driven, Test-After) |
| **Task-scoped** | unit is a plan task `T-04`, not an AC; same phases, same gate | Workflow 5 (PBV), Workflow 1 parallel mode |

### Test-after — the two extra rules

Writing the test after the code loses the "if I can't write the test, the spec is vague"
signal and invites tests that merely describe what the code happens to do. So:

1. **Write each test from the AC text**, never by reading the implementation.
2. **Mutate-and-check**: once the test passes, break the code path it covers (flip a
   condition, return early), confirm the test fails, then restore. A test that still passes
   against broken code is asserting nothing — rewrite it before the gate.

### Task-scoped

Substitute the plan's task id for the AC id everywhere: `test(T-04)`, `feat(T-04)`,
`Plan: docs/plans/<plan>.md#T-04`. The gate text says "T-04 is green (2/9 tasks done)".
Everything else is identical.

### Parallel builds

When implementation is fanned out to agents (SDD 5b-par, PBV parallel mode, Workflow 1
mode 2), the loop **batches**:

1. Agents implement a batch of independent ACs concurrently — one file per agent, conflict
   check first.
2. Merge, wire imports, run the full suite.
3. Then run **one gate per AC, in AC order**, over the merged result.

Concurrency changes when the code is written, never the order or the number of gates.

---

## 7. Exit

The loop ends when every AC is `✔`. Then, once for the whole feature:

- **Refactor** across ACs (the cleanups that were too big for a single gate).
- **Review + smoke** — the whole-feature review still runs; it simply has less to find.
- **ADR** for any non-obvious decision.
- Definition of Done, which now also requires: every AC has a commit pair (or its
  `<commit-style>` equivalent), and every gate was answered.
