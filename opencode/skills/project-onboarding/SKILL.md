---
name: project-onboarding
description: >
  Parallel codebase onboarding with one focused question upfront. Spawns 5 agents
  simultaneously via opencode's task tool (or an optional MCP swarm like Ruflo), builds a
  knowledge graph, writes a portable AGENTS.md (+ optional CLAUDE.md that imports it)
  and docs/PROJECT.md, generates Mermaid diagrams,
  detects toolchain (lint/CI/build), optionally persists findings to MCP memory, and onboards
  semantic-code MCPs if configured. Trigger on "onboard this project", "init project", "start a new project",
  "analyze this codebase", "what does this project do", "bootstrap sdd", "scaffold docs".
compatibility: opencode
---

# Project Onboarding

> Parallel codebase onboarding with one focused question upfront. Spawns 5 agents simultaneously, builds a knowledge graph, writes a portable AGENTS.md (+ an optional CLAUDE.md that imports it) and docs/PROJECT.md, persists findings to memory (if a memory MCP is configured), and onboards Serena (if configured).

## Trigger phrases

- "onboard this project"
- "init project"
- "start a new project"
- "analyze this codebase"
- "what does this project do"
- "bootstrap sdd"
- "scaffold docs"

---

## How this skill runs

Execute in this exact order:

---

### Step 0 — Ask one question

Ask the user this question and wait for their reply before proceeding:

> **What do you want to use this onboarding for?**
>
> 1. **Getting up to speed as a new contributor** — Focuses on how the codebase is structured, where things live, and how to run it locally. Best for someone joining the project for the first time.
> 2. **Planning a new feature or change** — Focuses on domain concepts, existing patterns, and which layers your change will touch. Best before writing a spec or starting implementation.
> 3. **Debugging / understanding existing behavior** — Focuses on entry points, data flow, and how components connect. Best when you need to trace a bug or understand why something behaves the way it does.
> 4. **Full audit** — Covers everything: structure, domain, patterns, tech debt, and gaps. Best for a deep review or before a major refactor.

Store their answer as `<onboarding-goal>` and use it to focus agent prompts in Step 3.

> **Just want a fast single-file pass?** This skill is the *deep* onboarding: parallel
> agents, knowledge graph, MCP memory, SDD scaffolding. If you only want a quick
> `CLAUDE.md`/`AGENTS.md` written from a single scan, use your agent's built-in
> **init/quick-scan** (e.g. Claude Code's `/init` or opencode's equivalent) instead — it's
> faster and purpose-built for that. Come back to this skill when you want the full picture.

---

### Step 1 — Create task list

Track progress with the todo tool (todowrite/todoread). Seed it with these tasks:

| # | Subject | activeForm |
|---|---------|------------|
| 1 | Init memory/swarm and Serena (optional) | Initializing |
| 2 | Run 5 parallel analysis agents | Analyzing codebase |
| 3 | Write docs/PROJECT.md | Writing PROJECT.md |
| 4 | Write AGENTS.md (+ optional CLAUDE.md) | Writing AGENTS.md |
| 5 | Build knowledge graph | Building knowledge graph |
| 6 | Persist to memory and Serena (optional) | Persisting to memory |
| 7 | Scaffold SDD structure | Scaffolding SDD |

---

### Step 2 — Init memory/swarm and Serena (fire and proceed, don't block)

If an MCP memory/swarm server (e.g. Ruflo) is configured, initialize it; otherwise skip — opencode runs subagents natively. When configured, send these calls in a **single message**. If any fail, proceed — never block:

```
mcp__ruflo__hooks_session-start({ sessionId: "<project-name>-onboarding" })
mcp__ruflo__hooks_init({})
mcp__ruflo__swarm_init({ topology: "mesh", maxAgents: 5 })
```

If a Serena MCP server is configured, onboard it (`mcp__serena__onboarding({})`); otherwise skip — rely on opencode's built-in code intelligence.

Derive `<project-name>` from the repo root folder name or the `name` field in `package.json` / `Cargo.toml` / `composer.json` / `pyproject.toml`.

Mark task 1 completed. Proceed immediately to step 3 regardless of result.

---

### Step 3 — Spawn 5 parallel agents (all in ONE message)

Spawn 5 subagents via opencode's `task` tool (run them concurrently) — send all five `task` calls in a **single message** so they run at the same time.
Append `<onboarding-goal>` context to each prompt so agents tailor their depth.

**Agent A — Architecture**
```
Analyze this codebase's technical architecture. Focus: <onboarding-goal>.
Return a structured report:
1. Architectural style (MVC, hexagonal, event-driven, monolith, microservices, serverless, etc.)
2. Key layers and what each does (controllers, services, repositories, use-cases, handlers, etc.)
3. Entry points (HTTP routes, CLI commands, queue consumers, cron jobs)
4. Where domain logic lives vs. infrastructure/framework code
5. Key patterns in use (DI, repository pattern, domain events, CQRS, sagas)
6. Top 3 tech debt signals (mixed patterns, logic in wrong layer, untested areas)
7. A compact Mermaid graph TD diagram — max 8 nodes, edges labelled with data flow direction.
   Format: ```mermaid\ngraph TD\n...```

Return only the structured report. No preamble.
```

**Agent B — Domain**
```
Analyze this codebase's business domain. Focus: <onboarding-goal>.
Return a structured report:
1. What this project does in plain language (1-2 sentences max)
2. Who uses it and what problem it solves
3. Core domain concepts / entities (5-8 most important nouns in the codebase)
4. The 3-5 most important user flows or operations
5. External dependencies (APIs, third-party services, databases)

Return only the structured report. No preamble.
```

**Agent C — Toolchain**
```
Scan this codebase for its development toolchain. Return exact commands for:
1. Build (Makefile, package.json scripts, justfile, taskfile.yml, Cargo.toml, etc.)
2. Test — include how to run a single test file
3. Lint (.eslintrc, ruff.toml, .flake8, phpstan.neon, .golangci.yml, biome.json, etc.)
4. Format (prettier, black, gofmt, rustfmt, etc.)
5. Dev server / run locally
6. CI pipeline (.github/workflows/, .gitlab-ci.yml, Jenkinsfile) — list key stages
7. Pre-commit hooks (.pre-commit-config.yaml, husky, lefthook)

For each: exact command string, or "not detected". Return only the report. No preamble.
```

**Agent D — SDD check**
```
Check this codebase for spec-driven development scaffolding. Return a structured report:
1. Does docs/specs/ exist? List spec files.
2. Does docs/adr/ exist? List ADR files.
3. Does docs/api/ or openapi.yaml exist?
4. Does docs/conventions.md exist?
5. Does docs/glossary.md exist?
6. Is there ARCHITECTURE.md, README.md, AGENTS.md, or CLAUDE.md? Summarize key points.

Return only the report. No preamble.
```

**Agent E — Knowledge graph**
```
Analyze this codebase and extract a knowledge graph of its entities and relationships.

Return a JSON object with exactly this shape:
{
  "entities": [
    { "id": "short-kebab-id", "name": "Display Name", "type": "service|module|model|controller|repository|queue|database|external-api|cli-command|event", "description": "one line" }
  ],
  "edges": [
    { "from": "entity-id", "to": "entity-id", "relation": "calls|depends-on|inherits|stores-to|reads-from|publishes|subscribes-to|triggers" }
  ]
}

Rules:
- Include max 20 entities — only the most important ones
- Every entity in edges must exist in entities
- No duplicate edges
- Return only the raw JSON object. No markdown, no preamble.
```

Mark task 2 completed when all five agents return.

---

### Step 4 — Write docs/PROJECT.md

Create `docs/` if it doesn't exist. Synthesize all agent reports into `docs/PROJECT.md`:

```markdown
# PROJECT.md

> Generated by project-onboarding skill. Single source of truth for business context and technical architecture.
> Onboarding goal: <onboarding-goal>

## What this project does

{1-3 sentence plain-language summary from Agent B}

## Who uses it

{from Agent B: users + problem solved}

## Core domain concepts

{5-8 key entities/concepts from Agent B, each with a one-line description}

## Architecture

**Style:** {from Agent A}

**Layers:**
{table or bullet: layer → what it does, from Agent A}

**Entry points:**
{list from Agent A}

### High-level component map

{Mermaid diagram from Agent A — embed inline, do NOT create a separate file}

## Key user flows

{top 3-5 flows from Agent B, numbered, 1-2 sentences each}

## External dependencies

{from Agent B}

## Development commands

| Task | Command |
|------|---------|
| Build | {from Agent C} |
| Run tests | {from Agent C} |
| Run single test | {from Agent C} |
| Lint | {from Agent C} |
| Format | {from Agent C} |
| Start dev server | {from Agent C} |

## CI pipeline

{from Agent C}

## Tech debt signals

{top 3 from Agent A, bullet + one line each}

## SDD status

{from Agent D: which docs exist, which are missing}
```

Rules:
- No placeholder text — every field must have real content
- Mermaid diagram inline — no separate `.mmd` files
- If a field has no data, write "not detected" — never omit the field

Mark task 3 completed.

---

### Step 4b — Write AGENTS.md (portable facts), then optionally CLAUDE.md

Write the **portable, tool-neutral project facts to `AGENTS.md`** — these are read by any
agent (opencode, Claude Code, Cursor, Codex, …). opencode reads `AGENTS.md` natively. Then
offer a thin Claude-specific `CLAUDE.md` that imports it, so there is **one source of truth
and no drift**.

#### 1. Always write `AGENTS.md`

If `AGENTS.md` already exists, add missing sections only — do not overwrite existing content.
If it does not exist, create it:

```markdown
# AGENTS.md

Guidance for AI coding agents working in this repository.

## Commands

| Task | Command |
|------|---------|
| Build | {from Agent C} |
| Run all tests | {from Agent C} |
| Run single test | {from Agent C} |
| Lint | {from Agent C} |
| Format | {from Agent C} |
| Start dev server | {from Agent C} |

## Architecture

**Style:** {from Agent A}

**Layers:**
{bullet: layer → what it does}

**Entry points:** {from Agent A}

**Key patterns:** {from Agent A}

**Domain logic lives in:** {specific folder/layer from Agent A}
```

Rules:
- Real command strings only — omit any row where tool was "not detected"
- Keep tool-neutral — no Claude-only or MCP-specific content here
- Keep under 60 lines

#### 2. Ask whether to also create `CLAUDE.md`

Ask the user this question and wait for their reply (single choice):

> AGENTS.md written. Also create a CLAUDE.md that imports it (recommended if you use Claude Code)?
>
> 1. **Yes — thin CLAUDE.md that imports AGENTS.md** — CLAUDE.md = @AGENTS.md + a small Claude-only section (skills, MCP servers). One source of truth, no duplication.
> 2. **Skip — AGENTS.md only** — opencode reads AGENTS.md directly, so no wrapper file is needed. (Claude Code also reads AGENTS.md.)

If **Yes** — write `CLAUDE.md` as a thin wrapper. If `CLAUDE.md` already exists, add the
import + missing Claude-only lines only; never duplicate the facts already in `AGENTS.md`:

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in this repository.

@AGENTS.md

## Claude Code notes

**Skills to use:** {e.g. /spec-driven-development for features, /clarify before any task, /workflow for debugging}
**MCP servers:** {e.g. Ruflo (memory + swarm), Serena (symbols) — if detected/onboarded}
**Gotchas:** {Claude-specific pitfalls, if any}
```

> `@AGENTS.md` is a Claude Code import — other tools (including opencode) read `AGENTS.md`
> directly and ignore this wrapper. So keep every universal fact in `AGENTS.md`; put only
> Claude-only lines here.

Mark task 4 completed.

---

### Step 5 — Build knowledge graph

Write `docs/knowledge-graph.json` using Agent E's JSON output exactly as returned.

Then render a summary table to the user:

```
Knowledge graph: {N} entities, {M} edges
Top entity types: {type: count, type: count, ...}
```

Mark task 5 completed.

---

### Step 6 — Persist to memory and Serena (optional)

If an MCP memory server (e.g. Ruflo) is configured, persist these entries; otherwise skip
persistence (the docs files are the durable output). When configured, send all calls in a
**single message** (parallel) and use `upsert: true`.

**Memory server entries (4):**

```
mcp__ruflo__memory_store({
  key: "<project-name>:domain", namespace: "project", upsert: true,
  value: { summary, users, coreConcepts, keyFlows, externalDeps }
})

mcp__ruflo__memory_store({
  key: "<project-name>:architecture", namespace: "project", upsert: true,
  value: { style, layers, entryPoints, patterns, techDebt }
})

mcp__ruflo__memory_store({
  key: "<project-name>:toolchain", namespace: "project", upsert: true,
  value: { build, test, singleTest, lint, format, devServer, ci }
})

mcp__ruflo__memory_store({
  key: "<project-name>:knowledge-graph", namespace: "project", upsert: true,
  value: { entities: [...], edges: [...] }   // Agent E output
})
```

**Serena memory (2 entries):** if a Serena MCP server is configured, write these memories;
otherwise skip — rely on opencode's built-in code intelligence.

```
mcp__serena__write_memory({
  memory_name: "project/domain",
  content: "# Domain\n\n{summary from Agent B}\n\n## Core concepts\n{concepts list}\n\n## Key flows\n{flows list}"
})

mcp__serena__write_memory({
  memory_name: "project/architecture",
  content: "# Architecture\n\n**Style:** {style}\n\n## Layers\n{layers}\n\n## Entry points\n{entry points}\n\n## Patterns\n{patterns}"
})
```

Mark task 6 completed.

---

### Step 7 — Scaffold SDD structure if missing

Read Agent D's report. Create only what's missing:

- **Missing `docs/specs/`** → create `docs/specs/.gitkeep`
- **Missing `docs/adr/`** → create `docs/adr/.gitkeep`
- **Missing `docs/conventions.md`** → create with real stack (Agent C) and layer names (Agent A)
- **Missing `docs/glossary.md`** → create with domain terms from Agent B's core concepts

If everything exists, skip and note "SDD structure already in place".

Mark task 7 completed.

---

### Step 8 — Done

Print:
```
Onboarding complete.
  AGENTS.md ✓
  CLAUDE.md ✓ (if chosen — imports AGENTS.md)
  docs/PROJECT.md ✓
  docs/knowledge-graph.json — {N} entities, {M} edges
  Memory — 4 entries stored (if a memory MCP is configured)
  Serena memory — 2 entries stored (if Serena is configured)
  SDD — {N} files scaffolded
```

> **Want a faster, single-file pass instead?** Use your agent's built-in init/quick-scan
> (e.g. Claude Code's `/init` or opencode's equivalent). This skill is intentionally the deep
> path — parallel agents, knowledge graph, MCP memory, and SDD scaffolding.

---

## What NOT to do

- Do NOT skip Step 0 — the one question shapes all agent analysis
- Do NOT run agents sequentially — all 5 must fire in one message
- Do NOT generate multiple diagram files — one Mermaid diagram, inline in PROJECT.md
- Do NOT leave template placeholders in any output file
- Do NOT block on memory/swarm or Serena init failures — always proceed
- Do NOT skip writing the docs files — every run must produce the durable outputs (persist to MCP memory only if configured)
- Do NOT create a separate knowledge graph diagram file — `docs/knowledge-graph.json` only

---

## Fallback: no memory/swarm MCP

opencode runs subagents natively via the `task` tool — spawn agents A–E directly with no
swarm server required. If no MCP memory server is configured, skip the memory persistence
steps; write Serena memories only if Serena is configured. The docs files (AGENTS.md,
docs/PROJECT.md, docs/knowledge-graph.json) are always the durable output.

---

## Relationship to other skills

| Next step | Skill |
|-----------|-------|
| Add a feature with SDD discipline | `spec-driven-development` |
| Deep parallel feature build | `multi-agent` |
| Code review | `code-review` |
