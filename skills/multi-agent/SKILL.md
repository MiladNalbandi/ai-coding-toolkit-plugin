# Multi-Agent — Parallel Work via Ruflo Swarm & Subagents

> Fan out independent work across multiple agents. Use this for codebase-wide analysis, parallel feature builds, adversarial review, and large migrations. Backed by Ruflo's `swarm_init` + `agent_spawn` MCP tools, or Claude's native Agent tool when Ruflo is unavailable.

## Trigger phrases

- "spawn agents"
- "run in parallel"
- "fan out"
- "multi-agent review"
- "swarm this"
- "parallel analysis"

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

**Ruflo version:**
```
ruflo swarm_init --topology mesh --max-agents 4
ruflo agent_spawn --role architect --task "analyze architecture"
ruflo agent_spawn --role domain-expert --task "extract domain"
ruflo agent_spawn --role auditor --task "find tech debt"
ruflo agent_spawn --role devops --task "detect tooling"
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

**Use git worktrees** (`superpowers:using-git-worktrees`) to prevent file conflicts.

---

### Pattern 4 — Large migration sweep

Discover sites, transform each in parallel, verify.

```
Migrate all callers of {{old API}} to {{new API}}.

Phase 1 — Discover: one agent greps for all call sites
Phase 2 — Transform: pipeline each call site through (read → rewrite → test)
Phase 3 — Verify: one agent runs the full test suite + smoke tests

Use Workflow tool with pipeline() for phase 2.
```

---

## Ruflo MCP integration

When Ruflo MCP is available, prefer it over Claude's native Agent tool because it provides:

- **Memory persistence** — `memory_store` shares context across agents
- **Hook routing** — `hooks_route` lets agents trigger each other
- **Swarm topology** — mesh, star, or pipeline coordination
- **Cross-session continuity** — swarms persist across sessions

### Setup

```bash
# 1. Verify Ruflo is initialized
ruflo init

# 2. Init a swarm with the right topology
ruflo swarm_init --topology mesh --max-agents 5

# 3. Spawn agents with roles
ruflo agent_spawn --role <role> --task "<task description>"

# 4. Query swarm status
ruflo swarm_status
```

### Topology choice

| Topology | When to use |
|----------|-------------|
| `mesh` | All agents need to share findings (analysis, review) |
| `star` | One coordinator delegates to N workers (feature fan-out) |
| `pipeline` | Output of agent N feeds agent N+1 (migration phases) |

---

## Fallback — Claude native Agent tool

If Ruflo is not installed, use Claude's `Agent` tool with `subagent_type`:

```
- Explore — fast read-only search agent for locating code
- Plan — architect agent for designing implementation plans
- general-purpose — catch-all for multi-step research/work
- claude — default catch-all
```

For true parallelism, send multiple `Agent` tool calls in a single message — they run concurrently.

---

## Workflow tool — deterministic orchestration

For complex multi-agent flows, use the `Workflow` tool:

- `parallel(thunks)` — barrier: wait for ALL agents
- `pipeline(items, stage1, stage2, ...)` — each item flows through stages independently
- `agent(prompt, opts)` — spawn one agent with schema validation

Default to `pipeline()`. Only use `parallel()` when stage N truly needs all of stage N-1's results.

---

## Tips

- Always tell each agent **explicitly what their return value should be** — agents are told their final text IS the data, not a human message
- Use **structured schemas** with `agent({schema})` to force valid JSON
- Use **worktree isolation** when agents mutate the same files in parallel
- Set **clear termination criteria** for adversarial loops (e.g. "stop after 2 dry rounds")
- Log dropped work with `log()` — silent truncation reads as "covered everything" when it didn't
