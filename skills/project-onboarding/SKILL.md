# Project Onboarding

> Run this at the start of any new project session to bootstrap context, tooling, and domain knowledge in one shot.

## Trigger phrases

- "onboard this project"
- "init project"
- "start a new project"
- "analyze this codebase"
- "what does this project do"

---

## Onboarding Prompt

Paste this at the start of a new project session:

```
Analyze this project and perform the following:

1. **Read the codebase** — scan structure, entry points, key files, and dependencies
2. **Initialize Ruflo MCP** — run `ruflo init` and wire up memory/hook tools
3. **Initialize RTK** — verify `rtk` is active and token savings are tracked
4. **Build a knowledge graph** — map modules, data flows, and key relationships using `memory_store`
5. **Business domain summary** — explain in plain language what this project does, who it serves, and its core value proposition
6. **Context report** — show current token usage breakdown (`/context`)
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
| Context report | `/context` | Shows token budget: system prompt, tools, memory, free space |

---

## Tips

- Run this **before** any feature work — it prevents wasted tokens on re-discovery later
- The knowledge graph built here is reused by `coding-workflows` and `spec-driven-development`
- If Ruflo MCP is not installed, skip step 2 and proceed — the rest still works
