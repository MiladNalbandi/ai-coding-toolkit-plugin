# Workflow 1: Feature Development

> **For rigorous, traceable feature work, use the `spec-driven-development` skill instead of this lightweight flow.** It enforces a spec → contract → red-tests → implement → Definition-of-Done loop with numbered acceptance criteria. The steps below are the fast path for small changes.
>
> Rule: if the change has acceptance criteria worth numbering, or touches validation / authorization / an API contract → switch to `spec-driven-development`. Otherwise the steps below are enough.

## Step 1 — Clarify requirements first

**Never write code before confirming scope.** Run the `clarify-loop` skill for your task type.

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

## Step 2 — Explore the codebase

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

## Step 3 — Design before coding

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

## Step 4 — Implement incrementally

First, ask the user:

```
How do you want to implement this? (default: 2)

  1. Sequential  — one layer at a time in this context (safe, simple)
  2. Parallel    — fan out independent layers to parallel agents (faster)
```

### Sequential (mode 1)

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

### Parallel (mode 2)

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

See the `multi-agent` skill for topology and fallback detail.

---

## Step 5 — Write tests

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

## Step 6 — Review pass

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
