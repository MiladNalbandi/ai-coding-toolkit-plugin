# AI Coding Toolkit — Claude Code Plugin

> Structured workflows, clarify-first loops, a prompt library, and MCP setup for fast AI-assisted development. Inspired by [obra/superpowers](https://github.com/obra/superpowers).

---

## Install

```bash
# Claude Code
claude-code plugin install https://github.com/{{your-username}}/ai-coding-toolkit

# Or add manually to your claude.json
{
  "plugins": [
    "https://github.com/{{your-username}}/ai-coding-toolkit"
  ]
}
```

---

## Skills

| Skill | Trigger | What it does |
|-------|---------|--------------|
| `spec-driven-development` | Building a feature "the right way", writing a spec or ADR | The rigorous backbone: spec → contract → red tests → implement → Definition of Done, with numbered ACs and stack mapping across PHP/Python/Go/Node/Rust |
| `coding-workflows` | Starting a feature, debug session, or review | Step-by-step workflow with prompts; hands off to `spec-driven-development` for non-trivial features |
| `clarify-loop` | Before any coding task | Surfaces the right questions, produces **numbered acceptance criteria**, and carries a Definition of Done checklist |
| `prompt-library` | Need a ready-made prompt | 12+ prompt templates across implement, debug, review, test, arch, ops |
| `mcp-toolkit` | Setting up or using MCP tools | How to wire up and use filesystem, browser, GitHub, and database MCPs |

### How the feature path flows across skills

```
clarify-loop          spec-driven-development              mcp-toolkit
(turn request    →    (spec → contract → red tests    +    (filesystem reads code,
 into numbered         → implement → refactor →             db runs EXPLAIN, browser
 acceptance            review → Definition of Done           reads live API docs while
 criteria)             → ADR)                                you implement)

coding-workflows owns the Debug, Architecture, and Review flows that SDD does not cover.
prompt-library supplies the copy-paste prompts for every step above.
```

---

## Usage

### Via slash commands (Claude Code)

```
/ai-coding-toolkit:coding-workflows
/ai-coding-toolkit:clarify-loop
/ai-coding-toolkit:prompt-library
/ai-coding-toolkit:mcp-toolkit
```

### Via natural language

The skills activate automatically based on context:
- "Let's build X" → `clarify-loop` first, then `coding-workflows`
- "Fix this bug" → `coding-workflows` (debug workflow)
- "How do I ask about performance?" → `prompt-library`
- "Set up the GitHub MCP" → `mcp-toolkit`

---

## Philosophy

These skills are inspired by Superpowers' core insight: **AI coding agents don't lack capability — they lack discipline.** The toolkit enforces:

1. **Clarify before coding** — never start with ambiguous requirements
2. **Design before implementing** — get a plan approved, then build it
3. **Incremental implementation** — one chunk at a time, each verifiable
4. **Tests as exit criteria** — done means tested, not just written
5. **MCP-first context** — let the AI read your actual code instead of pasting

---

## Relationship to Superpowers

This toolkit is **complementary to Superpowers**, not a replacement:

| Superpowers | This toolkit |
|-------------|-------------|
| Enforces TDD, brainstorming, subagent coordination | Focuses on clarify loops, prompt templates, MCP wiring |
| Rigid gates (must follow exactly) | Flexible guidance (adapt to your context) |
| Works across Claude Code, Cursor, Codex, Gemini | Designed for Claude Code + Claude.ai |

**Recommended:** Install both. Use Superpowers for the hard enforcement (TDD, debugging methodology), and this toolkit for the prompt templates and MCP setup patterns.

---

## Files

```
ai-coding-toolkit/
├── .claude-plugin/
│   └── metadata.json
├── skills/
│   ├── spec-driven-development/  # The rigorous feature-building backbone
│   │   ├── SKILL.md              #   spec → contract → red tests → DoD loop
│   │   ├── assets/               #   spec, ADR, conventions, glossary, openapi templates
│   │   └── references/           #   workflow, stack-mapping, database, frontend
│   ├── coding-workflows/
│   │   └── SKILL.md              # Feature, Debug, Architecture, Review flows
│   ├── clarify-loop/
│   │   └── SKILL.md              # 6 task types → numbered ACs + Definition of Done
│   ├── prompt-library/
│   │   └── SKILL.md              # 12+ ready-to-use prompt templates
│   └── mcp-toolkit/
│       └── SKILL.md              # Filesystem, Browser, GitHub, Database setup
└── README.md
```

---

## License

MIT — use, modify, share freely.
