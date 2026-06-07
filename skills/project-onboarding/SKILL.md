# Project Onboarding

> Run this at the start of any new project session to bootstrap context, tooling, domain knowledge, architecture understanding, and SDD scaffolding in one shot.

## Trigger phrases

- "onboard this project"
- "init project"
- "start a new project"
- "analyze this codebase"
- "what does this project do"

---

## Step 0 — Initialize Claude itself (do this before pasting the prompt)

Run these commands manually in a new Claude Code session before anything else:

```bash
# 1. Navigate to the project
cd /path/to/your/project

# 2. Set the model (Sonnet for most work, Opus for complex architecture)
/model

# 3. Set effort level
/effort

# 4. Verify plugins are loaded
/plugins

# 5. Check context is clean
/context

# 6. Verify RTK is active
rtk gain

# 7. Verify Ruflo MCP tools are available
/mcp
```

**Checklist before running onboarding:**

- [ ] Model set (Sonnet 4.6 default, Opus for deep architecture work)
- [ ] Effort level set (`high` for onboarding — you want thorough analysis)
- [ ] Working directory is the project root
- [ ] `ai-coding-toolkit` plugin is enabled (`/plugins`)
- [ ] RTK active (`rtk gain` shows version)
- [ ] MCP tools visible (`/mcp` shows connected servers)
- [ ] Context is fresh (new session, not a resumed one with stale context)

---

## Onboarding Prompt

Paste this at the start of a new project session:

```
Analyze this project and perform the following steps in order:

1. **Read the codebase** — scan structure, entry points, key files, and dependencies
2. **Initialize Ruflo MCP** — run `ruflo init` and wire up memory/hook tools
3. **Initialize RTK** — verify `rtk` is active and token savings are tracked
4. **Build a knowledge graph** — map modules, data flows, and key relationships using `memory_store`
5. **Business domain summary** — explain in plain language what this project does, who it serves, and its core value proposition
6. **Architecture summary** — describe the architectural style (MVC, hexagonal, event-driven, monolith, microservices, etc.), identify key layers, boundaries, and patterns used. Note any ADRs already present in docs/adr/
7. **SDD bootstrap** — check if docs/specs/, docs/adr/, and docs/api/ exist. If not, scaffold them using the spec-driven-development skill. Fill in docs/conventions.md and docs/glossary.md with the project's actual stack, layer names, and domain terms
8. **Context report** — show current token usage breakdown (`/context`)
```

---

## What each step does

| Step | Tool | Purpose |
|------|------|---------|
| Read codebase | Read / Bash / Explore agent | Builds structural understanding before any work begins |
| Ruflo MCP init | `ruflo init` + `swarm_init` | Activates memory store, hooks, and agent spawning |
| RTK init | `rtk gain` | Confirms token proxy is active; shows baseline savings |
| Knowledge graph | `memory_store` | Persists module map, data flows, and key relationships for future sessions |
| Business domain | Claude reasoning | Plain-language summary of what the project does and who it serves |
| Architecture summary | Claude reasoning + file scan | Identifies architectural style, layers, boundaries, patterns, and existing ADRs |
| SDD bootstrap | `spec-driven-development` skill | Scaffolds docs/specs/, docs/adr/, docs/api/, conventions.md, glossary.md — filled with the project's real stack and domain terms, not placeholder values |
| Context report | `/context` | Shows token budget: system prompt, tools, memory, free space |

---

## Architecture summary — what to look for

When analyzing architecture, Claude should identify and report:

- **Style** — MVC, hexagonal/ports-and-adapters, clean architecture, event-driven, CQRS, monolith, microservices, serverless
- **Layers** — what are they called in this project? (controllers, services, repositories, use-cases, handlers, etc.)
- **Entry points** — HTTP routes, CLI commands, queue consumers, cron jobs
- **Boundaries** — where does the domain end and infrastructure begin?
- **Key patterns** — dependency injection, repository pattern, domain events, sagas
- **Existing decisions** — any ADRs in docs/adr/, any ARCHITECTURE.md or similar
- **Tech debt signals** — mixed patterns, missing layers, logic in controllers, untested code

---

## SDD bootstrap — what gets created

If SDD scaffolding is missing, the onboarding creates:

```
docs/
├── sdd/00-workflow.md        # workflow reference, adapted to this project's stack
├── specs/                    # one NNN-feature.md per feature (starts empty)
├── adr/                      # immutable decision records (starts empty or imports existing)
├── api/openapi.yaml          # contract stub (HTTP projects only)
├── conventions.md            # filled with the project's real language, tools, layer names
└── glossary.md               # filled with the project's real domain terms
```

**Rules:**
- Never leave template placeholder values — replace with the project's actual stack
- Use `references/stack-mapping.md` to translate abstract roles to this project's idioms
- If an ARCHITECTURE.md or existing ADR already exists, import its decisions into docs/adr/ as ADR-0001, ADR-0002, etc.
- If the stack is unclear, ask once before scaffolding

---

## Tips

- Run this **before** any feature work — it prevents wasted tokens on re-discovery later
- The knowledge graph and SDD scaffold built here are reused by `coding-workflows` and `spec-driven-development`
- If Ruflo MCP is not installed, skip step 2 and proceed — the rest still works
- If SDD scaffolding already exists, step 7 becomes a **validation pass** — check that conventions.md and glossary.md reflect the current stack, not stale placeholders
