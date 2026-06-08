---
name: clarify-loop
description: >
  Clarifying questions to answer BEFORE writing any code, for six task types: new
  feature, bug fix, performance, refactor, API integration, and architecture. Produces
  numbered acceptance criteria and carries a Definition of Done checklist. Use at the
  very start of any task to remove ambiguity. Trigger on "before I start", "help me
  plan", "what should I ask", "I want to build or fix X but am not sure where to begin".
compatibility: opencode
---

# Clarify Loop — Ask Before You Code

> **Task:** $ARGUMENTS


> **The rule:** never start any coding task without answering these questions first. Ambiguity at the start costs 10x more than clarity.

Answer the questions for your task type below, then use the assembler template to build a complete prompt to paste into your AI coding session.

---

## Step 0 — Ask the user which clarify track to run

Before any clarification, ask the user **two questions** and wait for answers.

### Question 1 — Which task type?

Ask the user this question and wait for their reply. Present these choices as a single-select list:

- **New feature** — Building something that does not exist yet.
- **Bug fix** — Something is broken — find and fix the root cause.
- **Performance** — Too slow, hits a deadline, or costs too much.
- **Refactor** — Change shape without changing behavior.
- **API integration** — Wire up an external service.
- **Architecture** — Irreversible decision — pick an approach.

Store as `<task-type>`. Skip the question blocks for the other 5 types.

### Question 2 — Which sections do you want?

Ask the user this question and wait for their reply. This is a multi-select — the user can pick any number of the following sections:

1. **Requirement questions** — Walk through a task-specific checklist (scope, affected files, constraints, edge cases). Skip if requirements are already written and unambiguous.
2. **Acceptance criteria (AC-001, AC-002 …)** — Numbered, testable success conditions — each AC maps to a test case. Skip if you already have ACs from a spec.
3. **Definition of Done** — Exit checklist (tests pass, lint passes, PR reviewed, etc.) so 'done' means the same to you and the AI. Skip for trivial 1-line fixes.
4. **Red-flag check** — Scan for stop-the-line signals: hidden scope, unclear ownership, untestable ACs, missing auth decisions. Skip if scope is clean.
5. **Ready-to-send prompt** — A complete prompt with task + ACs + file list you can paste into a new session. Skip if you're handing off to a spec or plan.
6. **Next-step recommendation** — Recommend the next skill (coding-workflows / SDD / brainstorming / debugging) and what context to bring. Skip if you already know what's next.

Default behavior: if the user replies without selecting anything, treat that as "run all 6 sections". Use the user's reply to determine which to skip.

After collecting the answers, confirm before running:

```
Got it. Running:

  ✅ <included sections>
  ⬜ <skipped sections>

Starting now…
```

Store selected sections as `<active-sections>`. Run only those in order.

### Question 2.5 — Identity vs. authentication probe (auto-trigger)

**Trigger:** if the task description contains any of: `user`, `users`, `auth`, `login`,
`register`, `sign in`, `sign up`, `account`, `member`, `role`, `permission` — ask this
question BEFORE producing ACs:

```
Your task mentions "{{matched word}}". Before I write the acceptance criteria, I need
to understand what "users" means here — these are two very different things:

  A) **Named entities only** — a `users` table with name/email, no login flow.
     Tasks are linked to a user record. No session, no token, no password.
     → Small scope. Adds a FK, a user creation endpoint, maybe a list endpoint.

  B) **Authentication** — users can log in, get a token/session, and their identity
     gates what they can see or do. Password storage, token lifecycle, auth middleware.
     → Large scope. Adds auth layer, protected routes, session management.

  C) **Authorization only** — auth already exists, you want to control who can do what
     (roles, ownership checks, permission rules).
     → Medium scope. No new login flow, just guard existing routes.

Which is it — A, B, C, or a mix? If unsure, describe what a user should be able to DO
that they cannot do today.
```

Store answer as `<identity-scope>`. Use it to:
- Set the AC scope (A → FK + CRUD, B → full auth ACs, C → permission ACs)
- Flag in the spec: "User means {{A/B/C}} — see AC-U-001 for boundary"
- If B: recommend handoff to `superpowers:brainstorming` first (auth is a large, non-trivial design decision)

---

### Question 3 — Do you want the ACs reviewed before proceeding?

After producing numbered acceptance criteria (section 2), ask the user this question and wait for their reply. This is a multi-select:

- **Human review** — Pause and wait for your approval. You'll see the full AC list and can pick approve / edit / reject.
- **AI review (superpowers:requesting-code-review)** — Adversarial reviewer scans for vague/untestable ACs, overlap, missing edge cases, multi-requirement bullets.
- **Skip review** — Proceed straight to handoff. Use when you've already reviewed the ACs yourself.

**For Human review**, after showing the ACs ask the user this question and wait for their reply. This is a single-select decision:

- **✅ Approve — proceed to handoff** — ACs cover the intent. Continue to next-step recommendation.
- **✏️ Edit specific ACs** — I'll ask which AC numbers, apply changes, then re-ask.
- **🤖 Run AI review first** — Get adversarial second opinion before deciding.
- **❌ Reject — re-run requirement questions** — ACs don't match intent. Go back to Section 1 with new context.

**For AI review**, scan for:
- ACs that aren't testable ("works well", "user-friendly")
- ACs that overlap or contradict each other
- Missing edge cases (empty input, auth failures, rate limits)
- ACs that hide multiple requirements in one bullet

For each finding, ask the user this question and wait for their reply. Present per finding as a single-select (`AC-{{NNN}}: {{summary}}`):

- **✅ Apply — rewrite this AC** — Apply the suggested rewrite to make the AC testable / split / cover the edge case.
- **📋 Defer — add to follow-up** — Track as a follow-up AC for the next iteration.
- **❌ Reject — reviewer is wrong** — Keep the AC as-is. The reviewer misread the intent.

Loop until the AI reviewer returns no new findings, then hand off to `coding-workflows` or `spec-driven-development`.

---

## Task-type question blocks

Once `<task-type>` is chosen in Step 0, open **[`references/task-types.md`](references/task-types.md)**
and run only the matching block. Each block has:

- **Questions to answer** — a task-specific checklist (scope, affected files, constraints, edge cases).
- **Assembled prompt template** — a complete prompt to paste into a new AI coding session.

| `<task-type>` | Block |
|---------------|-------|
| New feature | Task Type 1 |
| Bug fix | Task Type 2 |
| Performance | Task Type 3 |
| Refactor | Task Type 4 |
| API integration | Task Type 5 |
| Architecture | Task Type 6 |

Do not load the whole reference — jump to the one block that matches.

---

## The Meta-Rule

Before ANY coding task, ask yourself these five questions. If you can't answer them, get the answers from your team or product owner first — not from the AI.

| Question | Why it matters |
|----------|----------------|
| What exactly am I building? | Prevents scope creep and building the wrong thing |
| Where does it live? | Avoids architectural misfits |
| What pattern should I follow? | Keeps the codebase consistent |
| What must not break? | Prevents regressions |
| How will I know it's done? | Defines a clear exit condition |

---

## Red Flags — Stop and Clarify if You Notice These

- You are guessing what the user/product owner actually wants
- The feature touches more than 3 services or modules
- You're not sure which of two patterns to follow
- The requirements contain the word "later", "eventually", or "maybe"
- You're about to change a public interface or database schema
- The task feels bigger than 1 day of work without a plan


---

## Definition of Done (adopted from spec-driven-development)

A change is mergeable only when **all** of these hold:

- [ ] Requirements captured as **numbered acceptance criteria** (AC-001, AC-002, …), each observable and testable.
- [ ] Tests exist, map back to ACs by name, and **failed before** implementation (red-first).
- [ ] Input validation is explicit and tested.
- [ ] Authentication, authorization, and **ownership** rules are explicit and tested.
- [ ] Sensitive/internal fields are not exposed; error responses are intentional.
- [ ] For HTTP features: the API contract was updated **before** the code and matches real behavior.
- [ ] Formatter, static analysis, full test suite, and a smoke test all pass.
- [ ] No behavior outside the agreed ACs was added (**YAGNI**).
- [ ] Any non-obvious decision has an ADR.

> The full discipline behind this checklist lives in the `spec-driven-development` skill.
