# Workflow 3: Architecture

## Step 1 — Define constraints

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

## Step 2 — Generate options

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

## Step 3 — Analyze tradeoffs

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

## Step 4 — Write the ADR

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

> The `spec-driven-development` skill ships a fuller ADR template at `assets/adr-template.md`.

---

## Step 5 — Phase the implementation

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
