# ai-coding-toolkit-plugin — Project Overview

> A Claude Code plugin that brings structured coding discipline into every session:
> five skills covering the full development lifecycle from clarification through
> spec-driven implementation, debugging, prompt templates, and MCP tooling.

---

## Overall Architecture

```mermaid
flowchart TD
    DEV([Developer / AI agent]) --> CL

    subgraph PLUGIN["ai-coding-toolkit-plugin"]
        direction TB

        CL["**clarify-loop**\n`/clarify`\nAsk before coding.\nProduces numbered ACs + DoD"]

        CL -->|simple task,\nno API contract| WF
        CL -->|complex feature,\nneeds spec + contract| SDD

        subgraph FEATURE["Feature path"]
            direction LR
            SDD["**spec-driven-development**\n`/spec-driven-development`\nspec → contract → red tests\n→ implement → DoD"]
        end

        subgraph DAILY["Daily workflows"]
            direction LR
            WF["**coding-workflows**\n`/workflow`\nFeature · Debug\nArchitecture · Review"]
        end

        SDD --> PR
        WF --> PR

        PR["**prompt-library**\n`/prompts`\n12+ templates:\nimplement · debug · review\ntest · arch · ops"]

        PR --> MC

        MC["**mcp-toolkit**\n`/mcp-setup`\nFilesystem · Browser\nGitHub · Database"]
    end

    MC --> TOOLS

    subgraph TOOLS["Real tools via MCP"]
        direction LR
        FS["filesystem\nread/write\ncodebase"]
        BR["browser\nlive docs\nDevTools"]
        GH["GitHub\nissues · PRs\nblame"]
        DB["database\nEXPLAIN\nANALYZE"]
    end

    style PLUGIN fill:#f8f8f8,stroke:#ccc,stroke-width:1px
    style FEATURE fill:#EEEDFE,stroke:#AFA9EC,stroke-width:1px
    style DAILY fill:#E1F5EE,stroke:#5DCAA5,stroke-width:1px
    style TOOLS fill:#E6F1FB,stroke:#85B7EB,stroke-width:1px
```

---

## Developer Workflow

```mermaid
flowchart LR
    START([Start any task]) --> CLARIFY

    CLARIFY{"/clarify\nWhat am I building?"}

    CLARIFY -->|"answers unclear,\nscope ambiguous"| Q["Ask 5 questions →\nnumbered ACs\n+ Definition of Done"]
    Q --> DECISION

    CLARIFY -->|"scope clear"| DECISION

    DECISION{Complex?\nAPI contract?\nAuth/validation?}

    DECISION -->|Yes| SDD["/spec-driven-development\nspec → contract\nred tests → implement\nreview → ADR"]
    DECISION -->|No| WF["/workflow\npick task type"]

    WF --> WF_TYPE{Task type}
    WF_TYPE -->|feature| FEA["Feature Dev\n6 steps"]
    WF_TYPE -->|bug| DBG["Debug\n5 steps"]
    WF_TYPE -->|design| ARC["Architecture\n5 steps"]
    WF_TYPE -->|check| REV["Code Review\n4 steps"]

    FEA & DBG & ARC & REV --> PROMPT["/prompts\nget the right template"]
    SDD --> PROMPT

    PROMPT --> MCP["/mcp-setup\nwire up tools"]
    MCP --> DONE([Implement with\nreal context])

    style SDD fill:#EEEDFE,stroke:#AFA9EC
    style CLARIFY fill:#FAEEDA,stroke:#FAC775
    style PROMPT fill:#E6F1FB,stroke:#85B7EB
    style MCP fill:#E1F5EE,stroke:#5DCAA5
```

---

## Plugin Structure

```mermaid
graph LR
    ROOT["ai-coding-toolkit-plugin/"]

    ROOT --> MANIFEST[".claude-plugin/\nplugin.json"]
    ROOT --> README["README.md"]
    ROOT --> SKILLS["skills/"]

    SKILLS --> S1["onboard-project/\nSKILL.md\nproject.md"]
    SKILLS --> S2["clarify-loop/\nSKILL.md"]
    SKILLS --> S3["coding-workflows/\nSKILL.md"]
    SKILLS --> S4["prompt-library/\nSKILL.md"]
    SKILLS --> S5["mcp-toolkit/\nSKILL.md"]
    SKILLS --> S6["spec-driven-development/\nSKILL.md\nassets/\nreferences/"]

    S6 --> A1["assets/\nspec-template.md\nadr-template.md\nopenapi-stub.yaml\n..."]
    S6 --> A2["references/\nworkflow.md\nstack-mapping.md\n..."]

    style ROOT fill:#f8f8f8,stroke:#ccc
    style S6 fill:#EEEDFE,stroke:#AFA9EC
    style S1 fill:#FAEEDA,stroke:#FAC775
```

---

## Skill Reference

| Command | Skill | Trigger | What it does |
|---|---|---|---|
| `/onboard` | `onboard-project` | First session, "explain this project" | Orients the developer, maps the project, produces a start plan |
| `/clarify [task]` | `clarify-loop` | Before any coding task | Asks the 5 questions, outputs numbered ACs + DoD |
| `/workflow [task]` | `coding-workflows` | Feature/debug/arch/review | Step-by-step workflow with prompts for every stage |
| `/spec-driven-development [feature]` | `spec-driven-development` | Complex feature, API contract, auth rules | Full spec → contract → red tests → implement → DoD loop |
| `/prompts [type]` | `prompt-library` | Need a ready-made template | 12+ copy-paste templates across implement/debug/review/test/arch/ops |
| `/mcp-setup [tool]` | `mcp-toolkit` | Wiring up MCP servers | Filesystem, browser, GitHub, and database MCP setup + key prompts |

---

## Skill Dependency Map

```mermaid
graph TD
    ON["onboard-project\n/onboard"]

    ON -->|"first, always"| CL["clarify-loop\n/clarify"]

    CL -->|"simple tasks"| WF["coding-workflows\n/workflow"]
    CL -->|"complex features"| SDD["spec-driven-development\n/spec-driven-development"]

    WF -->|"needs a prompt template"| PL["prompt-library\n/prompts"]
    SDD -->|"needs a prompt template"| PL

    WF -->|"needs real codebase context"| MC["mcp-toolkit\n/mcp-setup"]
    SDD -->|"needs live API docs or DB"| MC
    PL -->|"needs real tools to execute"| MC

    SDD -.->|"borrows DoD + ACs"| CL

    style SDD fill:#EEEDFE,stroke:#AFA9EC
    style CL fill:#FAEEDA,stroke:#FAC775
    style MC fill:#E1F5EE,stroke:#5DCAA5
    style PL fill:#E6F1FB,stroke:#85B7EB
    style ON fill:#FCEBEB,stroke:#F09595
```

---

## MCP Integration Map

```mermaid
flowchart LR
    subgraph AGENT["Claude Code session"]
        WF2["coding-workflows"]
        SDD2["spec-driven-development"]
        DBG2["debug workflow"]
    end

    subgraph MCP_SERVERS["MCP servers"]
        FS2["filesystem\n~/.claude/skills\n.claude/ project files"]
        BR2["Claude in Chrome\nlive docs\nDevTools network tab"]
        GH2["GitHub MCP\nissues · PRs · blame\nsearch codebase"]
        DB2["Database MCP\nEXPLAIN ANALYZE\ndata inspection"]
    end

    WF2 -->|"read/write files"| FS2
    SDD2 -->|"read existing code\nwrite generated files"| FS2
    SDD2 -->|"read live API docs\nverify contract"| BR2
    DBG2 -->|"run EXPLAIN ANALYZE\ninspect data rows"| DB2
    WF2 -->|"find similar implementations\nread issue context"| GH2

    style AGENT fill:#f8f8f8,stroke:#ccc
    style MCP_SERVERS fill:#E1F5EE,stroke:#5DCAA5
```

---

## Installation

```bash
# Personal — available in every project
unzip ai-coding-toolkit-plugin.zip
cp -r ai-coding-toolkit-plugin ~/.claude/skills/

# Project-scoped — shared with team via version control
cp -r ai-coding-toolkit-plugin .claude/skills/

# Or as a GitHub marketplace plugin
/plugin marketplace add MiladNalbandi/ai-coding-toolkit-plugin
/plugin install ai-coding-toolkit@ai-coding-toolkit-plugin
```

Verify installation:

```
/help          ← your custom commands should appear here
/onboard       ← start here in any new session
/doctor        ← check Claude Code health + plugin status
```

---

## Design Principles

| Principle | How it's implemented |
|---|---|
| Clarify before coding | `/clarify` is always the first step; produces numbered ACs |
| Traceability | Numbered ACs (AC-001, AC-002…) chain from clarify → tests → code |
| Contract-first | `spec-driven-development` enforces OpenAPI/proto before implementation |
| Tests before code | SDD enforces red-first; tests named after ACs |
| MCP-first context | Skills reference MCP prompts so Claude reads actual files, not pastes |
| Composable | Each skill is independent but cross-references neighbours |

---

## Relationship to Superpowers

This plugin is **complementary to** [obra/superpowers](https://github.com/obra/superpowers):

| obra/superpowers | ai-coding-toolkit-plugin |
|---|---|
| TDD enforcement, brainstorming gates, subagent coordination | Clarify loops, prompt library, MCP wiring |
| Hard gates — rigid methodology | Flexible — adapt to context |
| Feature-building discipline | Full lifecycle: clarify · build · debug · review · MCP |

Install both:

```bash
/plugin marketplace add obra/superpowers
/plugin marketplace add MiladNalbandi/ai-coding-toolkit-plugin
```

---

## Contributing

1. Fork → branch → edit a `SKILL.md`
2. Test by dropping the folder into `~/.claude/skills/` and running the command
3. Open a PR with before/after examples of the skill output

The `spec-driven-development` skill (`/spec-driven-development`) is the authoritative
source for contribution workflow — use it to spec any changes to this plugin itself.
