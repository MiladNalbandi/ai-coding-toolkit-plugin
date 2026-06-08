# ai-coding-toolkit — opencode port

The same 7 skills as the Claude Code plugin, ported to [opencode](https://opencode.ai).

opencode reads **Claude-compatible `SKILL.md` files natively** (same `name` / `description`
frontmatter and `references/` + `assets/` progressive disclosure), so these are real
opencode skills — not a reimplementation. The only changes from the Claude version are
**tool mechanics** (see the mapping table below): structured questions, todos, subagents,
and MCP memory are expressed in opencode terms.

## The 7 skills

| Skill | What it does |
|-------|--------------|
| `clarify-loop` | Ask the right questions first → numbered acceptance criteria |
| `coding-workflows` | Feature / debug / architecture / review / plan-build-verify playbooks, with explicit coding + testing structure (unit → integration → e2e) |
| `spec-driven-development` | Spec → contract → red tests → implement → Definition of Done |
| `project-onboarding` | Parallel subagents → portable `AGENTS.md` (+ optional `CLAUDE.md`), knowledge graph, docs |
| `prompt-library` | 12+ ready-made prompt templates |
| `mcp-toolkit` | Wire up filesystem / browser / GitHub / database MCP servers (via `opencode.json`) |
| `multi-agent` | Fan out independent work across parallel subagents |

## Install

opencode discovers skills from several locations. Pick one:

**Project-scoped** (this repo only) — copy or symlink into `.opencode/skills/`:

```bash
mkdir -p .opencode
cp -R /path/to/ai-coding-toolkit-plugin/opencode/skills .opencode/skills
# or symlink to track upstream:
ln -s /path/to/ai-coding-toolkit-plugin/opencode/skills .opencode/skills
```

**Global** (all projects) — copy into `~/.config/opencode/skills/`:

```bash
mkdir -p ~/.config/opencode/skills
cp -R /path/to/ai-coding-toolkit-plugin/opencode/skills/* ~/.config/opencode/skills/
```

**Already using the Claude plugin?** opencode also reads `~/.claude/skills/` and
`.claude/skills/` directly — so if the Claude Code plugin is installed, opencode can use
those skills as-is (with Claude-specific tool calls). This `opencode/` port exists to give
you the opencode-native tool wiring instead.

### Verify

Start opencode and the skills appear via the native `skill` tool. opencode invokes a skill
with `skill({ name: "clarify-loop" })`, or you ask in natural language and it triggers from
the description — e.g. *"help me clarify this feature before I build it."*

## Tool mapping (Claude Code → opencode)

| Claude Code | opencode |
|-------------|----------|
| `AskUserQuestion` (structured picker) | Plain question + numbered options; agent waits for the reply |
| `TaskCreate` / `TaskUpdate` / `TaskList` | `todowrite` / `todoread` (the todo tool) |
| native `Agent` tool (parallel calls) | the `task` tool; custom subagents live in `.opencode/agents/` |
| `mcp__ruflo__*` (swarm / memory) | **optional** — opencode runs subagents natively via `task`; MCP swarm/memory used only if configured |
| `mcp__serena__*` (semantic code) | **optional** MCP — else opencode's built-in LSP + grep |
| MCP setup in `settings.json` / `.mcp.json` | the `mcp` key in `opencode.json` (`type: "local"` + command, or `type: "remote"` + url) |
| `superpowers:*` skills | tool-neutral phrasing of what they do (adversarial review, evidence-before-done, TDD loop, etc.) |

Anything in the skill bodies that depends on Ruflo or Serena is written as **optional** —
the skills work with opencode's built-in tools and only *use* those MCP servers if you've
configured them.

## Portable by design: `AGENTS.md`

`project-onboarding` writes a tool-neutral **`AGENTS.md`** (project facts: commands,
architecture, layers, entry points) that **opencode reads natively** — and so do Cursor,
Codex, and others. It then optionally adds a thin `CLAUDE.md` that imports it (`@AGENTS.md`)
for anyone who also uses Claude Code. One source of truth, no drift.

## License

MIT — same as the parent plugin.
