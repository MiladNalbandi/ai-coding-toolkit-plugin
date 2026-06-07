# Prompt Library — Ready-to-Use AI Coding Templates

> Copy, fill in the `{{placeholders}}`, and send. Organized by task category.

---

## IMPLEMENT

### Feature scaffold

```
Task: implement {{feature name}}
Location: {{service / module}}
Stack: {{language, framework}}
Follow pattern: {{existing file to mirror}}

Before any code:
1. Confirm your understanding of requirements
2. List every file you will create or modify
3. Wait for my approval
Then implement one file at a time.
```

---

### Extend existing code

```
I need to extend {{class / function}} to support {{new behavior}}.

Existing behavior to preserve: {{what it does now}}
New behavior needed: {{what to add}}
Interface constraint: {{what callers expect}}

Approach:
1. Summarize the current implementation so we agree on behavior
2. Show me only the diff (not full rewrite unless necessary)
3. Explain every non-trivial decision
```

---

### API endpoint

```
Create a {{GET|POST|PUT|DELETE}} {{/path}} endpoint.

Purpose: {{what it does}}
Input: {{request body / query params}}
Output: {{response shape}}
Auth: {{who can call this}}
Errors to handle: {{list}}

Follow the existing controller pattern in {{file}}.
Include: input validation, proper HTTP status codes, error response format.
```

---

## DEBUG

### General bug diagnosis

```
Bug: {{what is happening}}
Expected: {{what should happen}}
Stack: {{language + framework + versions}}
Environment: {{local / staging / prod}}

Error / stack trace:
```
{{paste here}}
```

Recent changes: {{what changed}}
Already tried: {{what you ruled out}}

Diagnose systematically. Rank hypotheses by likelihood.
For each: give me the cheapest test to confirm or deny it.
```

---

### Async / concurrency bug

```
Concurrency issue in {{component}}.

Symptom: {{what happens — race condition, deadlock, data corruption?}}
Concurrency model: {{threads / coroutines / message queue / multi-pod}}
When it happens: {{frequency, pattern — always? only under load?}}

```
{{relevant code}}
```

Questions to guide your analysis:
- Is there shared mutable state? Where?
- Are there non-atomic read-modify-write sequences?
- Could the order of message processing matter here?
- Is there a missing lock, transaction, or idempotency guard?
```

---

### Slow query investigation

```
Slow query investigation.

Query / endpoint: {{describe}}
Current: {{latency}} at {{row count / request rate}}
Target: {{acceptable latency}}

EXPLAIN ANALYZE output:
```
{{paste here}}
```

ORM / raw SQL:
```
{{paste query}}
```

Identify: missing index, N+1, full scan, join explosion, or missing pagination.
Propose fixes ranked by impact. Show the rewritten query.
```

---

## REVIEW

### Full code review

```
Review this {{language}} code. Be direct — list issues by severity.

Context: {{what this code does, who calls it}}

```
{{paste code}}
```

Check in this order:
1. Blockers: logic errors, missing null checks, security holes
2. Warnings: performance, missing error handling, bad naming
3. Suggestions: style, testability, over-engineering

For each blocker/warning: show the bad line + the fixed version.
```

---

### Security audit

```
Security audit for this code.

Context: this handles {{type of data}}, called by {{who/what}}.

```
{{paste code}}
```

OWASP checklist:
- SQL / command / path injection
- Broken authentication or authorization
- Sensitive data in logs or error responses
- Unsafe deserialization
- Missing input validation
- Secrets in code or env vars

Rate: critical / high / medium / low.
For each finding: exploit scenario + fix.
```

---

## TEST

### Unit test generation

```
Write unit tests for {{class / function}}.
Framework: {{PHPUnit / pytest / jest / ...}}
Language: {{language}}

```
{{paste code to test}}
```

Cover:
1. Happy path
2. Edge cases: null input, empty collections, boundary values
3. Error cases: invalid input, external dependency failure
4. Any important invariants

Follow the test style in {{existing test file}}.
Each test: independent, descriptive name, one assertion per concept.
```

---

### Characterization tests (legacy code)

```
Write characterization tests for this legacy code before I refactor it.

Goal: capture the CURRENT behavior exactly, even if wrong.
These tests must pass before AND after my refactoring.

```
{{paste legacy code}}
```

Framework: {{framework}}

For each behavior you identify:
1. Write a test that pins it
2. Note if the behavior seems intentional or accidental
3. Flag anything that looks like a bug I should ask about
```

---

## ARCHITECTURE

### Architecture decision

```
I need to decide: {{decision to make}}

Context:
- System: {{what we're building / changing}}
- Stack: {{current tech}}
- Team: {{size + skills}}
- Timeline: {{deadline}}

Options I'm considering:
A) {{option A}}
B) {{option B}}

Constraints:
{{hard limits}}

Walk me through the tradeoffs. Ask clarifying questions first.
End with a clear recommendation + the key assumption it depends on.
```

---

### Refactoring plan

```
I need to refactor {{system / module}} from {{current state}} to {{target state}}.

Why: {{reason — maintainability, performance, new requirements}}
Risk: {{what could break}}
Team: {{engineers, timeline}}

Constraints:
- Must stay deployable throughout
- {{other hard constraints}}

Give me a phased plan where:
- Each phase ships independently
- Phase 1 proves the riskiest assumption
- No phase leaves the system broken
```

---

## OPS

### Kubernetes / container debug

```
Container issue in {{service name}}.

Symptom: {{CrashLoopBackOff / OOMKilled / ImagePullBackOff / other}}
Environment: {{cluster, namespace}}

kubectl describe output:
```
{{paste here}}
```

Recent logs:
```
{{paste last 50 lines}}
```

Diagnose. Check: resource limits, liveness/readiness probes,
config/secret mounting, image issues, startup ordering.
Give me the exact kubectl commands to investigate further.
```

---

### CI/CD pipeline failure

```
CI/CD failure in {{pipeline / step name}}.

Failure: {{what failed}}
Pipeline tool: {{GitHub Actions / GitLab CI / Jenkins / etc}}

Failing step output:
```
{{paste here}}
```

Recent changes to pipeline config: {{yes/no, what}}
Does it fail consistently or flakily? {{answer}}

Diagnose and give me the fix. If the root cause is unclear,
tell me which logs or artifacts to collect next.
```

---

## QUICK START EXAMPLES

### Debug a bug

```
I have a bug: {{describe what's wrong and where}}.
Walk me through the debug workflow.
```

### Fix a bug

```
Fix this bug: {{paste error or describe behavior}}.
File: {{path}}, Line: {{number if known}}.
```

### Add a small feature

```
Add {{feature}}. Here's the context: {{describe}}.
```

### Add a non-trivial feature (recommended)

```
I want to add {{feature}}. Run clarify-loop first, then spec-driven-development.
```

This forces:
1. Clarify requirements → numbered acceptance criteria
2. Write spec + ADR
3. Define API contract
4. Red tests first
5. Implement
6. Definition of Done checklist

### Golden rule — always start here

```
Before we start, help me confirm the requirements for: {{task}}
```

---

## Tips for Better Prompts

### Do

- **Be specific about what already exists** — paste the relevant file or describe the existing pattern
- **State constraints upfront** — must-nots are as important as requirements
- **Paste the full error** — not just the last line, the whole stack trace
- **Give the AI permission to ask questions** — "ask me if anything is unclear before coding"
- **Set an explicit approval gate** — "wait for my go-ahead before implementing"

### Don't

- Ask for "the best way" without context — there is no best without constraints
- Describe what you want in terms of implementation — describe the behavior, let the AI pick the implementation
- Paste 500 lines and ask for a review — narrow the scope or ask for the most critical issues only
- Accept the first solution — ask "what are the tradeoffs of this approach?"
- Skip error messages — "it doesn't work" is not a bug report

---

## Prompt Escalation Ladder

If a prompt gives a poor result, escalate in this order:

1. **Add more context** — paste the relevant existing code
2. **Be more specific** — replace vague words (good, clean, proper) with concrete requirements
3. **Add few-shot examples** — show what you want with a "like this" example
4. **Add chain-of-thought** — "think through this step by step before answering"
5. **Split the task** — if one prompt does two things, split it into two prompts
6. **Add negative examples** — "do NOT do X, I've already tried that"
