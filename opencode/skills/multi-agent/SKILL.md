---
name: multi-agent
description: >
  Fan out independent work across multiple agents via opencode's task tool (or an
  optional MCP swarm backend like Ruflo). Use for codebase-wide analysis, parallel feature builds,
  adversarial review, and large migrations. Trigger on "spawn agents", "run in parallel",
  "fan out", "multi-agent review", "swarm this", "parallel analysis".
compatibility: opencode
---

# Multi-Agent — Parallel Work via Subagents (with optional Ruflo Swarm)

> Fan out independent work across multiple agents. Use this for codebase-wide analysis, parallel feature builds, adversarial review, and large migrations. Backed by opencode's native `task` tool, or an optional MCP swarm backend (like Ruflo's `swarm_init` + `agent_spawn`) when one is configured.

## Trigger phrases

- "spawn agents"
- "run in parallel"
- "fan out"
- "multi-agent review"
- "swarm this"
- "parallel analysis"

---

## How agent spawning works in opencode

opencode spawns subagents via the **`task` tool**. Each `task` invocation runs a subagent with its own prompt and context; the subagent's final text is returned to the orchestrator.

- **Custom subagents** can be defined in `.opencode/agents/` (project-level) or `~/.config/opencode/agents/` (global). Reference them by name when invoking `task`.
- **Concurrency**: opencode runs multiple `task` invocations concurrently. To fan out, issue several `task` calls together so they execute in parallel rather than one after another.
- **Optional swarm backend**: If an MCP swarm server like Ruflo is configured, you can use its `swarm_init` / `agent_spawn` tools with a chosen topology (see below). Otherwise, opencode's native `task` tool is the default and requires no extra setup.

---

## When to use multi-agent

**Use it when:**
- 2+ tasks are genuinely independent (no shared state, no sequential dependencies)
- You need adversarial review (one agent builds, another tries to break it)
- The codebase is too large for a single context window
- A migration touches many files with the same transform

**Do NOT use it when:**
- Tasks depend on each other's output
- A single agent can do the work in one pass cheaply
- The task is small enough that orchestration overhead dominates

---

## Patterns

These patterns are backend-agnostic: use opencode's native `task` tool by default, or an MCP swarm backend (like Ruflo) if one is configured.

### Pattern 1 — Parallel onboarding analysis

Split codebase analysis across agents during project onboarding.

```
Spawn 4 parallel agents to analyze this codebase:

Agent A — Architecture: identify style, layers, boundaries, entry points
Agent B — Domain: extract business concepts, user stories, value proposition
Agent C — Tech debt: find mixed patterns, missing tests, untyped code
Agent D — Tooling: detect lint, formatter, CI, test runner, build system

Each agent returns a structured report. Synthesize into a single onboarding doc.
```

**opencode (native):** issue 4 `task` invocations together (one per agent) so they run concurrently, then synthesize the returned reports.

**Optional swarm backend (e.g. Ruflo):**
```
swarm_init --topology mesh --max-agents 4
agent_spawn --role architect --task "analyze architecture"
agent_spawn --role domain-expert --task "extract domain"
agent_spawn --role auditor --task "find tech debt"
agent_spawn --role devops --task "detect tooling"
```

---

### Pattern 2 — Adversarial code review

One agent builds, N agents try to break it.

```
After implementing {{feature}}:

Spawn 3 reviewer agents with different lenses:
- Reviewer A — Correctness: try to find logic bugs and edge cases
- Reviewer B — Security: try to find injection, auth bypass, data leaks
- Reviewer C — Performance: try to find N+1, memory leaks, blocking calls

Each reviewer is told to default to "refuted=true" if uncertain. A finding survives only if ≥2 reviewers agree it's real.
```

In opencode, issue the 3 reviewer `task` calls together so they review concurrently.

---

### Pattern 3 — Parallel feature build

Multiple independent features ship at once.

```
I have 3 independent tasks:
1. Add user profile endpoint
2. Add password reset flow
3. Add email verification

These don't share state. Spawn 3 agents in worktree isolation, one per task. Each follows the full SDD loop. Synthesize results into 3 PRs.
```

**Use git worktrees** to prevent file conflicts — give each agent its own worktree (or otherwise isolated working copy) so parallel writes never collide. This advice is generic: any time agents mutate the same files concurrently, isolate them per worktree.

---

### Pattern 4 — Large migration sweep

Discover sites, transform each in parallel, verify.

```
Migrate all callers of {{old API}} to {{new API}}.

Phase 1 — Discover: one agent greps for all call sites
Phase 2 — Transform: pipeline each call site through (read → rewrite → test)
Phase 3 — Verify: one agent runs the full test suite + smoke tests
```

In opencode, run Phase 2 as a fan-out of `task` calls (one per call site, executed concurrently), with Phase 1 feeding the list and Phase 3 gating on all transforms completing. A swarm backend with a `pipeline` topology can model this directly (see below).

---

## Topology — conceptual coordination shapes

Whether you fan out with native `task` calls or an MCP swarm backend, it helps to pick a coordination shape:

| Topology | When to use |
|----------|-------------|
| `mesh` | All agents need to share findings (analysis, review) |
| `star` | One coordinator delegates to N workers (feature fan-out) |
| `pipeline` | Output of agent N feeds agent N+1 (migration phases) |

With native `task`, the orchestrator (you) acts as the hub: it collects each subagent's returned text and decides what to pass on. A swarm backend can enforce these topologies explicitly.

---

## Optional MCP swarm backend (e.g. Ruflo)

If an MCP swarm server like Ruflo is configured, you can use it instead of native `task` calls. A swarm backend can add:

- **Memory persistence** — a `memory_store`-style tool shares context across agents
- **Hook routing** — agents can trigger each other
- **Swarm topology** — explicit mesh, star, or pipeline coordination
- **Cross-session continuity** — swarms persist across sessions

### Setup (Ruflo example)

```bash
# 1. Verify the swarm backend is initialized
ruflo init

# 2. Init a swarm with the right topology
swarm_init --topology mesh --max-agents 5

# 3. Spawn agents with roles
agent_spawn --role <role> --task "<task description>"

# 4. Query swarm status
swarm_status
```

If no swarm backend is configured, skip all of this and use opencode's native `task` tool (the default).

---

## Optional MCP — semantic code tools (e.g. Serena)

If a semantic-code MCP like Serena is configured, agents can use its symbol-level search and editing tools for more precise navigation. This is optional; agents work fine with plain file reads and grep when it is absent.

---

## Native `task` orchestration — tips

For complex multi-agent flows using opencode's `task` tool, think in two shapes:

- **Barrier (parallel)** — wait for ALL fanned-out `task` calls to return before synthesizing. Issue them together, then collect every result.
- **Pipeline** — each item flows through stages independently (read → rewrite → test). Drive this from the orchestrator: feed each stage's output into the next.

Default to a pipeline. Only use a full barrier when stage N truly needs all of stage N-1's results.

---

## Tips

- Always tell each agent **explicitly what their return value should be** — a subagent's final text IS the data, not a human message
- Ask agents to return **structured output** (e.g. JSON in a fixed shape) when you need to parse and merge results deterministically
- Use **worktree isolation** when agents mutate the same files in parallel
- Set **clear termination criteria** for adversarial loops (e.g. "stop after 2 dry rounds")
- Log dropped work explicitly — silent truncation reads as "covered everything" when it didn't
