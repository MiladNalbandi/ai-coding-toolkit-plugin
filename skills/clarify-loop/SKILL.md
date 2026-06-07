---
name: clarify-loop
description: >
  Clarifying questions to answer BEFORE writing any code, for six task types: new
  feature, bug fix, performance, refactor, API integration, and architecture. Produces
  numbered acceptance criteria and carries a Definition of Done checklist. Use at the
  very start of any task to remove ambiguity. Trigger on "before I start", "help me
  plan", "what should I ask", "I want to build or fix X but am not sure where to begin".
command: /clarify
---

# Clarify Loop — Ask Before You Code

> **Task:** $ARGUMENTS


> **The rule:** never start any coding task without answering these questions first. Ambiguity at the start costs 10x more than clarity.

Answer the questions for your task type below, then use the assembler template to build a complete prompt to paste into your AI coding session.

---

## Step 0 — Ask the user which clarify track to run

Before any clarification, ask the user **two questions** and wait for answers.

### Question 1 — Which task type?

**Use `AskUserQuestion` (single-select)** so the user picks from a real menu:

```json
{
  "question": "What kind of task is this?",
  "header": "Task type",
  "multiSelect": false,
  "options": [
    { "label": "New feature", "description": "Building something that does not exist yet." },
    { "label": "Bug fix", "description": "Something is broken — find and fix the root cause." },
    { "label": "Performance", "description": "Too slow, hits a deadline, or costs too much." },
    { "label": "Refactor", "description": "Change shape without changing behavior." },
    { "label": "API integration", "description": "Wire up an external service." },
    { "label": "Architecture", "description": "Irreversible decision — pick an approach." }
  ]
}
```

Store as `<task-type>`. Skip the question blocks for the other 5 types.

### Question 2 — Which sections do you want?

**Use the `AskUserQuestion` tool with `multiSelect: true` to render a real interactive checklist.** Do NOT prompt with text and wait for typed numbers — use the tool so the user sees actual checkboxes they can tick.

### Tool call shape

```json
{
  "question": "Which sections should I run for this clarify-loop?",
  "header": "Sections",
  "multiSelect": true,
  "options": [
    {
      "label": "1. Requirement questions",
      "description": "Walk through a task-specific checklist (scope, affected files, constraints, edge cases). Skip if requirements are already written and unambiguous."
    },
    {
      "label": "2. Acceptance criteria (AC-001, AC-002 …)",
      "description": "Numbered, testable success conditions — each AC maps to a test case. Skip if you already have ACs from a spec."
    },
    {
      "label": "3. Definition of Done",
      "description": "Exit checklist (tests pass, lint passes, PR reviewed, etc.) so 'done' means the same to you and the AI. Skip for trivial 1-line fixes."
    },
    {
      "label": "4. Red-flag check",
      "description": "Scan for stop-the-line signals: hidden scope, unclear ownership, untestable ACs, missing auth decisions. Skip if scope is clean."
    },
    {
      "label": "5. Ready-to-send prompt",
      "description": "A complete prompt with task + ACs + file list you can paste into a new session. Skip if you're handing off to a spec or plan."
    },
    {
      "label": "6. Next-step recommendation",
      "description": "Recommend the next skill (coding-workflows / SDD / brainstorming / debugging) and what context to bring. Skip if you already know what's next."
    }
  ]
}
```

Default behavior: if the user presses Enter without ticking anything, treat that as "run all 6 sections". Use the returned `answers` map to determine which to skip.

After collecting the answers, confirm before running:

```
Got it. Running:

  ✅ <included sections>
  ⬜ <skipped sections>

Starting now…
```

Store selected sections as `<active-sections>`. Run only those in order.

**Why use `AskUserQuestion`:** the user sees real checkboxes, can multi-select with the keyboard or mouse, and can pick "Other" to override. Asking 6 sequential yes/no text prompts is poor UX and slow.

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

After producing numbered acceptance criteria (section 2), ask:

```
ACs are drafted. How do you want to review them? (default: human)

  [x] Human review        — pause and wait for your approval
  [x] AI review (superpowers:requesting-code-review) — adversarial check for vague,
                            untestable, or overlapping ACs
  [ ] Skip review         — proceed straight to handoff
```

For AI review, the reviewer should look for:
- ACs that aren't testable ("works well", "user-friendly")
- ACs that overlap or contradict each other
- Missing edge cases (empty input, auth failures, rate limits)
- ACs that hide multiple requirements in one bullet

Apply / defer / reject each finding. Loop until clean before handing off to
`coding-workflows` or `spec-driven-development`.

---

## Task Type 1: New Feature

### Questions to answer

1. **What needs to be built?** — One clear sentence. What is the feature?
2. **Which service / module?** — Where does it live in the codebase?
3. **Pattern to follow?** — Which existing file or class should this mirror?
4. **Tech stack** — Language, framework, key libraries.
5. **Requirements** — What must be true when this is done? (list each one)
6. **Constraints** — What must NOT change or break? Any performance / backward-compat limits?

### Assembled prompt template

```
I need to implement: {{feature description}}

Location: {{service / module}}  |  Stack: {{language, framework}}
Follow the pattern from: {{existing file or class}}

Requirements:
{{list each requirement}}

Constraints:
{{list must-nots and limits}}

---
Before writing any code:
1. Ask me if anything is unclear
2. Turn the requirements into NUMBERED acceptance criteria (AC-001, AC-002, …)
   — each observable, testable, and small enough to map to one test.
3. List every file you will touch and the key design decisions
4. Wait for my approval
Then implement one step at a time, naming tests after the AC they cover.
```

> **Upgrade path:** the moment you write numbered ACs, you're doing spec-driven development. For anything non-trivial, save these ACs into `docs/specs/NNN-feature.md` and run the `spec-driven-development` loop.

---

## Task Type 2: Fix a Bug

### Questions to answer

1. **Symptom** — What is happening? Be specific about the wrong behavior.
2. **Expected behavior** — What should happen instead?
3. **Steps to reproduce** — Minimal steps to trigger the bug.
4. **Error message / stack trace** — Paste the full error.
5. **Recent changes** — What changed recently? (deploys, config, new data patterns)
6. **Already tried** — What have you ruled out? (Don't waste AI time on dead ends)

### Assembled prompt template

```
Bug Report

Symptom: {{what is happening}}
Expected: {{what should happen}}

Steps to reproduce:
{{numbered steps}}

Error / stack trace:
```
{{paste full error}}
```

Recent changes: {{what changed}}
Already tried: {{what you've ruled out}}

---
Diagnose this systematically.
Rank hypotheses by likelihood, then walk me through the cheapest test for each one.
```

---

## Task Type 3: Performance Optimization

### Questions to answer

1. **What is slow?** — Specific endpoint, query, or function.
2. **Current measured latency** — P50/P95/P99 or average. Use numbers.
3. **Target** — What is acceptable? Be specific.
4. **Stack + volume** — Language, DB, request rate, data size.
5. **Profiling data** — EXPLAIN ANALYZE output, APM trace, or flame graph (if available).

### Assembled prompt template

```
Performance issue: {{what is slow}}
Current: {{measured latency}}  →  Target: {{acceptable latency}}
Stack: {{language, DB, framework}}  |  Volume: {{req/min, row count}}

Profiling data:
```
{{EXPLAIN ANALYZE / APM trace / flame graph}}
```

---
Identify the bottleneck. For each optimization:
1. Where time is being spent (query / network / compute / serialization)
2. Top 2–3 changes ranked by impact vs effort
3. How to measure that each change actually helped
```

---

## Task Type 4: Refactoring

### Questions to answer

1. **What to refactor?** — File + class name + language.
2. **Goal** — What quality dimension to improve? (readability / testability / performance / structure)
3. **Public interface constraints** — What must NOT change from callers' perspective?
4. **Pattern to follow** — Existing example or architectural principle to apply.
5. **Test coverage** — Are there existing tests? Safe to refactor? Or need characterization tests first?

### Assembled prompt template

```
Refactor: {{file / class + language}}
Goal: {{what to improve and why}}

Public interface constraint: {{what callers expect — must not change}}
Follow pattern: {{existing file or architecture principle}}
Test situation: {{current coverage}}

---
Before refactoring:
1. Summarize what the current code does (so we agree on behavior to preserve)
2. Propose the target structure
3. Wait for my approval
Then refactor in small, behavior-preserving steps.
```

---

## Task Type 5: API Integration

### Questions to answer

1. **Which API?** — Name + docs URL if available.
2. **What do you need from it?** — Which endpoints and data.
3. **Auth mechanism** — API key, OAuth2, Basic Auth, etc.
4. **Your existing HTTP client pattern** — What to follow for consistency.
5. **Error handling requirements** — Retry logic, circuit breaker, alerting?

### Assembled prompt template

```
API Integration: {{API name}}
Docs: {{URL}}
Need: {{endpoints + data required}}
Auth: {{mechanism}}

Follow client pattern: {{existing file or class}}
Error handling: {{retry policy, circuit breaker, alerting requirements}}

---
Implement a clean integration client:
1. Follow my existing HTTP client pattern exactly
2. Handle all documented error codes with typed exceptions
3. Include request/response logging
4. Make auth injectable for testing
```

---

## Task Type 6: Architecture Decision

### Questions to answer

1. **What are you designing?** — System, feature, or migration.
2. **Key requirements** — Functional + non-functional (latency, scale, reliability).
3. **Existing stack + constraints** — What you must work within.
4. **Team + timeline** — Engineers available and deadline.
5. **Options you're considering** — Your current thinking (even rough).

### Assembled prompt template

```
Architecture: {{what you're designing}}
Stack: {{existing tech}}  |  Team: {{size + deadline}}

Requirements:
{{functional and non-functional requirements}}

Options I'm considering:
{{Option A: description}}
{{Option B: description}}

Constraints:
{{hard limits}}

---
Ask me clarifying questions first. Then:
1. Analyze tradeoffs of each option against my requirements
2. Recommend one with explicit reasoning
3. Identify the top 3 risks and how to mitigate them
```

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
