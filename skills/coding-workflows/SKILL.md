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


> Step-by-step processes for the four most common coding tasks. Follow the order. Never skip Clarify and Design — they prevent 80% of rework.

---

## Workflow 1: Feature Development

> **For rigorous, traceable feature work, use the `spec-driven-development` skill instead of this lightweight flow.** It enforces a spec → contract → red-tests → implement → Definition-of-Done loop with numbered acceptance criteria. The steps below are the fast path for small changes.
>
> Rule: if the change has acceptance criteria worth numbering, or touches validation / authorization / an API contract → switch to `spec-driven-development`. Otherwise the steps below are enough.

### Step 1 — Clarify requirements first

**Never write code before confirming scope.** Run the clarify loop (see `02-clarify-loop.md`) for your task type.

Prompt to use:
```
Before we start, help me confirm the requirements.

Feature: {{feature name}}

Questions I need answered:
1. What is the exact scope? What is explicitly OUT of scope?
2. Which existing code should I model this after?
3. What does "done" look like — how will we verify it?
4. Any performance, security, or backward-compat constraints?

Ask me these one by one if I haven't answered them above.
```

---

### Step 2 — Explore the codebase

Map out relevant files, interfaces, and patterns before designing anything.

```
Explore this codebase for implementing {{feature}}.

Find:
- The most relevant existing files / classes
- The naming conventions and patterns used
- The entry point (controller / command / handler) I should add to
- Any interfaces or contracts I must implement or respect

Summarize what you find before we design anything.
```

---

### Step 3 — Design before coding

Get a written plan approved. This is the highest-leverage step.

```
Based on what we've explored, propose an implementation plan for {{feature}}.

Include:
1. Files you will create or modify (with full paths)
2. Key classes / functions and their responsibilities
3. Any new interfaces or contracts introduced
4. How this integrates with existing code
5. What could go wrong and how you'll handle it

Wait for my approval before writing any implementation code.
```

---

### Step 4 — Implement incrementally

First, ask the user:

```
How do you want to implement this? (default: 2)

  1. Sequential  — one layer at a time in this context (safe, simple)
  2. Parallel    — fan out independent layers to parallel agents (faster)
```

#### Sequential (mode 1)

One logical chunk at a time. Each chunk must be runnable.

```
Implement step {{N}} of the plan: {{step description}}.

Rules:
- One file or one coherent unit at a time
- Show the complete file, not a diff
- Follow the existing patterns exactly (naming, error handling, logging)
- Add a comment if you make a non-obvious decision
- Stop and ask if you hit something unexpected

After each step, summarize what you did and what comes next.
```

#### Parallel (mode 2)

Map the approved plan's steps to agents — **one file per agent, no overlap**.

**Conflict check first:** list all files each agent will write. If any file appears
twice, serialize those two agents. Never let two agents share a file.

**If Ruflo is available:**
```
ruflo swarm_init --topology star --max-agents <N>
ruflo agent_spawn --role <layer> --task "Implement <step> per the approved plan. Write only <file>. Follow existing patterns in this codebase."
# repeat per layer
```

**If Ruflo unavailable — Claude native Agent tool:**
Send all `Agent` tool calls in a **single message** so they run concurrently.
Each agent prompt must include: the approved plan excerpt, the exact file to write,
the layer contract (inputs/outputs), and the YAGNI constraint.

**After all agents complete:**
1. Merge all outputs into the working tree
2. Wire imports between layers
3. Run tests — show pass/fail per test
4. Fix inline if any fail (don't re-spawn for small fixes)

---

### Step 5 — Write tests

Cover happy path + edge cases discovered during design.

```
Write tests for {{class/function}} using {{PHPUnit/pytest/jest}}.

Cover:
1. Happy path — normal successful execution
2. Edge cases: {{list the ones you know}}
3. Error cases: invalid input, external service failure, concurrent access
4. Any invariants that must always hold

Follow the test style in {{existing test file}}.
Each test: independent, descriptive name, one assertion per concept.
```

---

### Step 6 — Review pass

Structured review before committing. Catches 90% of issues before CI.

```
Review this implementation before I commit.

Check for:
- Logic errors and off-by-one mistakes
- Missing null/empty checks
- Security: SQL injection, path traversal, unsafe deserialization
- N+1 queries or unnecessary DB calls
- Missing error handling or swallowed exceptions
- Code that will be hard to understand in 6 months

Rate each issue: blocker / warning / suggestion.
List blockers first.
```

---

## Workflow 2: Debugging

### Step 1 — Reproduce reliably

You cannot debug what you cannot reproduce.

```
I'm debugging: {{bug description}}

Help me create a minimal reproduction.

Symptom: {{what happens}}
Environment: {{local/staging/prod}}, {{stack + versions}}
Full error / stack trace:
{{paste here}}

What is the smallest possible input that triggers this?
Can we isolate it to a single function call?
```

---

### Step 2 — Read the error carefully

Stack traces contain most of the answer. Parse them bottom-up.

```
Parse this error message / stack trace carefully:

```
{{paste full error here}}
```

Tell me:
1. The exact line where the exception was thrown
2. The call chain that led there (read the trace bottom-up)
3. What the error message literally means
4. What the code was trying to do when it failed
5. The most likely root cause layer: application / framework / infra / data
```

---

### Step 3 — Isolate the layer

Determine which layer owns the bug.

```
Help me isolate which layer owns this bug.

Symptom: {{what happens}}
Stack: {{tech stack}}

Layer checklist:
- Application logic: wrong condition, missing null check, bad state machine?
- Framework / library: unexpected behavior in version {{X}}?
- Infrastructure: network timeout, pod restart, memory pressure?
- Data: unexpected value in DB, encoding issue, null where non-null expected?

What's the fastest test I can run to rule each one in or out?
```

---

### Step 4 — Hypothesize and test

Form ranked hypotheses. Test cheapest-first, most-likely-first.

```
Help me rank my hypotheses.

What I've established so far:
{{your findings}}

For each hypothesis:
1. The specific prediction (what would be true if this is the cause)
2. The cheapest test to confirm or deny it (log, unit test, DB query)
3. Your confidence: high / medium / low

Order by: (likelihood × speed of test).
```

---

### Step 5 — Fix, verify, prevent

Minimal fix. Verify it. Add a test so it never silently regresses.

```
Root cause identified: {{your conclusion}}

Help me:
1. Write the minimal fix — change only what is necessary
2. Explain why this fixes the root cause (not just the symptom)
3. Identify similar places in the codebase with the same bug pattern
4. Write a test that would have caught this bug before it hit production
5. Suggest a monitoring alert or log statement to detect future recurrence
```

---

## Workflow 3: Architecture

### Step 1 — Define constraints

Architecture without constraints is just dreaming.

```
I'm designing: {{system or feature}}

Before generating options, list the hard constraints:

Technical: {{language, existing infra, must-use services}}
Performance: {{latency SLA, throughput, data volume}}
Team: {{engineers available, expertise, timeline}}
Operational: {{on-call burden, observability requirements}}
Business: {{must not break X, must support Y}}

Challenge any constraint that seems artificial.
```

---

### Step 2 — Generate options

Name 2–3 viable approaches. Resist converging too early.

```
For {{system/feature}}, generate 2–3 viable architecture options.

For each option:
- Name it clearly (e.g. "Event-sourced pipeline", "Polling worker pool")
- Describe the approach in 3–4 sentences
- List the key components
- Identify the core assumption it makes

Do NOT evaluate them yet. Just lay them out clearly.
```

---

### Step 3 — Analyze tradeoffs

Compare on YOUR constraints, not generic ones.

```
Compare these architecture options on dimensions that matter for my constraints.

Constraints: {{from step 1}}

For each option, evaluate:
- Complexity to build (effort × risk)
- Operational burden (monitoring, failure modes, on-call)
- Scalability headroom
- Team fit (can we maintain this?)
- Reversibility (how hard to undo if wrong?)

End with a clear recommendation and the key assumption it depends on.
```

---

### Step 4 — Write the ADR

Document the decision. Future teammates need to understand the why.

```
Write an Architecture Decision Record (ADR).

# ADR: {{title}}

## Status
Accepted

## Context
{{why this decision was needed}}

## Decision
{{what we decided}}

## Consequences
Positive: {{what improves}}
Negative: {{what we accept as a cost}}
Risks: {{what could go wrong}}

## Alternatives considered
{{other options and why rejected}}
```

---

### Step 5 — Phase the implementation

Break into phases you can ship and validate independently.

```
Break the implementation of {{architecture}} into phases.

Rules:
- Each phase must be independently deployable
- Each phase must leave the system in a working state
- Phase 1 should prove the riskiest assumption
- Later phases build on validated foundations

For each phase: name, goal, what gets built, how to validate, size (S/M/L)
```

---

## Workflow 4: Code Review

### Step 1 — Logic and correctness

```
Review this code for logic correctness.

```{{language}}
{{paste code}}
```

Check:
- Does the happy path work as described?
- Are all branches handled (null, empty, out-of-range)?
- Off-by-one errors in loops or slices?
- Race conditions if this runs concurrently?
- Is every error caught and handled appropriately?

Rate each issue: blocker / warning.
```

### Step 2 — Security scan

```
Security review for this code.

Context: this handles {{what kind of data / who calls it}}

Check for:
- Injection: SQL, command, path traversal
- Broken auth / missing authorization checks
- Sensitive data in logs or error messages
- Unsafe deserialization or type coercion
- Missing input validation and sanitization
- Secrets hardcoded or in environment variables

Flag everything, even if low-severity.
```

### Step 3 — Performance

```
Performance review for this code.

Context: called {{frequency}}, data volume: {{N rows / M req/min}}

Check:
- N+1 queries or missing eager loading
- Missing DB indexes for the query patterns used
- Large objects allocated in loops
- Synchronous blocking calls that could be async
- Missing caching for repeated expensive computations

Only flag issues relevant to the actual load this will handle.
```

### Step 4 — Maintainability

```
Maintainability review for this code.

Check:
- Function and variable names — do they describe intent?
- Function length — does each function do one thing?
- Is anything surprisingly complex for what it achieves?
- Missing or misleading comments (especially on non-obvious decisions)
- Duplication that should be extracted
- Test coverage — what's unverifiable in its current form?
- Is it over-engineered for the actual requirements?

Give concrete rewrite suggestions, not just flags.
```

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
