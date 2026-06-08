# Workflow 4: Code Review

Four focused passes. Run them in order — correctness first, polish last.

## Step 1 — Logic and correctness

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

## Step 2 — Security scan

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

## Step 3 — Performance

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

## Step 4 — Maintainability

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
