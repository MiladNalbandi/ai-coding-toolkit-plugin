# Workflow 2: Debugging

> For a deeper, evidence-driven investigation, also run a systematic debugging loop:
> reproduce → read the error → isolate the layer → form ranked hypotheses → test the
> cheapest-first → fix the root cause → add a regression test.

## Step 1 — Reproduce reliably

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

## Step 2 — Read the error carefully

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

## Step 3 — Isolate the layer

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

## Step 4 — Hypothesize and test

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

## Step 5 — Fix, verify, prevent

Minimal fix. Verify it. Add a test so it never silently regresses.

```
Root cause identified: {{your conclusion}}

Help me:
1. Write the minimal fix — change only what is necessary
2. Explain why this fixes the root cause (not just the symptom)
3. Identify similar places in the codebase with the same bug pattern
4. Write a **unit-level regression test** that would have caught this bug before production
5. Confirm the fixed code path is covered — run a coverage check on the touched file(s)
6. Suggest a monitoring alert or log statement to detect future recurrence
```

> A fix is not done until it has a failing-then-passing regression test. See
> [`testing-structure.md`](testing-structure.md) for the unit-test rules and coverage budget.
