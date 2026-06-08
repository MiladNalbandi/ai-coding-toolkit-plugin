# Clarify Loop — Task-Type Question Blocks

For each task type: the questions to answer, then an assembled prompt template you can
paste into a new AI coding session. Run **only** the block matching `<task-type>` from
Step 0.

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

> For a full architecture loop (constraints → options → tradeoffs → ADR → phasing), use Workflow 3 in the `coding-workflows` skill.
