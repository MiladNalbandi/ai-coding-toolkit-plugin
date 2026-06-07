# Project Onboarding

> Run this at the start of any new project session to bootstrap context, tooling, domain knowledge, architecture understanding, SDD scaffolding, optional diagrams, and (optionally) a Ruflo agent swarm — in one shot.

## Trigger phrases

- "onboard this project"
- "init project"
- "start a new project"
- "analyze this codebase"
- "what does this project do"
- "bootstrap sdd"
- "scaffold docs"

---

## Step 0 — Initialize Claude itself (do this before pasting the prompt)

Run these commands manually in a new Claude Code session before anything else:

```bash
cd /path/to/your/project
/model           # Sonnet 4.6 default, Opus 4.7 for deep architecture work
/effort          # Set to high for onboarding — you want thorough analysis
/plugins         # Verify ai-coding-toolkit is enabled
/context         # Confirm context is fresh
rtk gain         # Verify RTK token proxy is active
/mcp             # Verify Ruflo MCP and other servers are connected
```

**Pre-flight checklist:**

- [ ] Model set (Sonnet 4.6 default, Opus 4.7 for complex onboarding)
- [ ] Effort level: `high`
- [ ] Working directory is the project root
- [ ] `ai-coding-toolkit` plugin enabled
- [ ] RTK active
- [ ] MCP servers visible
- [ ] Context is fresh (new session, not resumed with stale context)

---

## Step 0.5 — Interactive setup questions

Before running the onboarding loop, Claude should ask the user:

1. **MCP servers** — "Which MCP integrations do you want to use? (ruflo, filesystem, github, database, browser, all, none)"
2. **Multi-agent mode** — "Do you want me to use a Ruflo swarm with parallel agents for analysis? (yes/no)"
3. **Diagrams** — "Do you want me to generate diagrams of the code architecture and business domain? (yes/no)"
4. **SDD scaffold** — "Should I scaffold SDD docs (docs/specs/, docs/adr/, docs/api/) if missing? (yes/no)"
5. **Lint/CI detection** — "Should I detect lint, formatter, and CI pipelines? (yes/no)"

Defaults if user says "use defaults": MCPs=ruflo only, swarm=yes, diagrams=yes, SDD=yes, lint=yes.

---

## Onboarding Prompt

Paste this at the start of a new project session:

```
Onboard this project. Run these steps in order:

**FIRST: Create a TaskCreate task list with one task per step below (steps 0–11). Mark each task `in_progress` when you start it and `completed` when done. This gives the user live visibility into onboarding progress.**

0. **Setup questions** — Ask me about MCPs, swarm mode, diagrams, SDD scaffold, lint detection (per Step 0.5 of the skill). Wait for answers.

1. **Read the codebase** — scan structure, entry points, key files, and dependencies

2. **Initialize selected MCPs** — based on my answers, init only what I chose:
   - Ruflo: run `ruflo init`, then `ruflo swarm_init --topology mesh --max-agents 4`
   - Filesystem MCP: confirm read access to project root
   - GitHub MCP: confirm token + repo access
   - Database MCP: confirm connection string
   - Browser MCP: confirm headless browser ready

3. **Initialize RTK** — verify `rtk` is active and token savings are tracked

4. **Multi-agent analysis (if swarm=yes)** — spawn 4 Ruflo agents in parallel via `ruflo agent_spawn`:
   - Agent A — Architecture: style, layers, boundaries, entry points
   - Agent B — Domain: business concepts, user stories, value proposition
   - Agent C — Tech debt: mixed patterns, untyped code, missing tests
   - Agent D — Tooling: lint, formatter, CI, test runner, build system
   Synthesize results into a single onboarding report.

   If swarm=no, do all four analyses sequentially in a single context.

5. **Build a knowledge graph** — persist module map, data flows, key relationships, and the synthesized analysis into `memory_store` so future sessions inherit it

6. **Business domain summary** — explain in plain language what this project does, who it serves, and its core value proposition

7. **Architecture summary** — describe architectural style (MVC, hexagonal, event-driven, monolith, microservices, etc.), identify key layers, boundaries, patterns. Note existing ADRs in docs/adr/

8. **Diagrams (if diagrams=yes)** — generate Mermaid diagrams:
   - **Architecture diagram** — components, layers, data flow between them
   - **Domain diagram** — entities, relationships, bounded contexts
   - **Sequence diagrams** — for the 2–3 most important user flows
   - **C4 model** (if the project is large) — Context, Container, Component levels
   Save all diagrams to `docs/diagrams/` as `.mmd` files plus a `README.md` index

9. **Lint & pipeline detection (if lint=yes)** — scan for and report:
   - Lint configs: .eslintrc, ruff.toml, .flake8, phpstan.neon, .golangci.yml, biome.json
   - Formatters: prettier, black, gofmt, rustfmt
   - Test runners: pytest, jest, phpunit, go test
   - CI: .github/workflows/, .gitlab-ci.yml, Jenkinsfile, circle.yml
   - Pre-commit: .pre-commit-config.yaml, husky, lefthook
   - Build: package.json scripts, Makefile, justfile, taskfile.yml
   For each: show the exact command to run it locally

10. **SDD bootstrap (if SDD=yes)** — check if docs/specs/, docs/adr/, docs/api/ exist. If not, scaffold them using the `spec-driven-development` skill. Fill conventions.md and glossary.md with the project's actual stack, layer names, and domain terms (no placeholder values)

11. **Context report** — show current token usage breakdown (`/context`)
```

---

## What each step does

| Step | Tool | Purpose |
|------|------|---------|
| 0 | Manual CLI + Claude commands | Configure session before onboarding |
| 0.5 | Claude interactive Q&A | Personalize the onboarding to user preferences |
| 1 | Read / Bash / Explore | Build structural understanding |
| 2 | `ruflo init`, `swarm_init`, MCP configs | Activate only the integrations user wants |
| 3 | `rtk gain` | Confirm token proxy |
| 4 | `ruflo agent_spawn` ×4 OR sequential | Parallel codebase analysis |
| 5 | `memory_store` | Persist knowledge graph |
| 6 | Claude reasoning | Business domain summary |
| 7 | Claude reasoning + scan | Architecture summary |
| 8 | Mermaid generation | Visual code + domain understanding |
| 9 | File scan | Surface existing lint/CI/build pipelines |
| 10 | `spec-driven-development` skill | SDD scaffolding |
| 11 | `/context` | Token budget snapshot |

---

## MCP options

| MCP | What it does | When to enable |
|-----|--------------|----------------|
| **ruflo** | Memory store, swarm, hooks, agent spawning | Always (foundation) |
| **filesystem** | Direct read/write to project files | Most projects |
| **github** | Read PRs, issues, releases, repo metadata | Projects with active GitHub workflow |
| **database** | Query DB schema, run EXPLAIN, sample data | Data-backed features |
| **browser** | Read live docs, scrape APIs, screenshot UI | Frontend or API integration work |

---

## Diagram options

When diagrams=yes, Claude generates:

| Diagram | Mermaid type | Use case |
|---------|--------------|----------|
| Architecture | `graph TD` or `C4Container` | High-level component map |
| Domain model | `classDiagram` or `erDiagram` | Entities + relationships |
| Sequence | `sequenceDiagram` | User flows (login, checkout, etc.) |
| State machine | `stateDiagram-v2` | For features with explicit states |
| Deployment | `graph LR` | Infra topology |

All diagrams saved to `docs/diagrams/<name>.mmd` with a `docs/diagrams/README.md` index.

---

## Architecture summary — what to look for

- **Style** — MVC, hexagonal, clean, event-driven, CQRS, monolith, microservices, serverless
- **Layers** — controllers, services, repositories, use-cases, handlers — whatever the project calls them
- **Entry points** — HTTP routes, CLI commands, queue consumers, cron jobs
- **Boundaries** — where domain ends and infrastructure begins
- **Key patterns** — DI, repository, domain events, sagas, CQRS
- **Existing decisions** — ADRs in docs/adr/, ARCHITECTURE.md
- **Tech debt signals** — mixed patterns, logic in controllers, untested code

---

## SDD bootstrap — what gets created

```
docs/
├── sdd/00-workflow.md
├── specs/
├── adr/
├── api/openapi.yaml
├── diagrams/
│   ├── README.md
│   ├── architecture.mmd
│   ├── domain.mmd
│   └── sequence-*.mmd
├── conventions.md
└── glossary.md
```

**Rules:**
- Never leave template placeholders — replace with the project's real stack
- Import any existing ARCHITECTURE.md or ADRs into docs/adr/
- If stack is unclear, ask once before scaffolding

---

## Task list template

When the onboarding starts, Claude should create these tasks via `TaskCreate`:

| # | Subject | activeForm |
|---|---------|------------|
| 1 | Ask setup questions (MCPs, swarm, diagrams, SDD, lint) | Asking setup questions |
| 2 | Read codebase structure | Reading codebase |
| 3 | Initialize selected MCPs | Initializing MCPs |
| 4 | Verify RTK token proxy | Verifying RTK |
| 5 | Run multi-agent analysis (or sequential) | Analyzing codebase |
| 6 | Persist knowledge graph to memory_store | Persisting knowledge graph |
| 7 | Write business domain summary | Summarizing domain |
| 8 | Write architecture summary | Summarizing architecture |
| 9 | Generate Mermaid diagrams | Generating diagrams |
| 10 | Detect lint, formatter, CI, build pipelines | Detecting toolchain |
| 11 | Scaffold or validate SDD docs | Scaffolding SDD |
| 12 | Show context report | Reporting context usage |

Tasks 9, 10, 11 are skipped if the user opted out in step 0.

---

## Tips

- Run this **before** any feature work — it prevents wasted tokens on re-discovery later
- The knowledge graph + SDD scaffold + diagrams are reused by `coding-workflows`, `spec-driven-development`, and `multi-agent` skills
- If Ruflo MCP is missing, swarm steps fall back to Claude's native `Agent` tool with `subagent_type=Explore` or `general-purpose`
- If SDD scaffolding already exists, step 10 becomes a **validation pass** — check that conventions.md and glossary.md reflect the current stack
- Diagrams should be **regenerated** when major architectural decisions land (after each new ADR)
