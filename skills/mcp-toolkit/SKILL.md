---
name: mcp-toolkit
description: >
  How to set up and use MCP servers for coding — filesystem, browser (Claude in Chrome),
  GitHub, and database — with key prompts and workflow integration for each. Use when
  wiring up MCP tools or deciding which MCP to use for a coding task. Trigger on
  "set up MCP", "use the filesystem or github or database tool", "which MCP should I use".
command: /mcp-setup
---

# MCP Toolkit — AI Coding with Real Tools

> **MCP task:** $ARGUMENTS


> MCPs (Model Context Protocol servers) let your AI coding assistant read files, browse docs, query databases, and interact with GitHub — without you pasting anything manually.

---

## Why MCPs change how you code with AI

Without MCPs, you paste code → AI reads it → AI responds → you manually apply changes.

With MCPs, AI reads your actual files, browses live docs, runs queries against your DB, and writes back to your filesystem. The feedback loop shrinks from minutes to seconds.

---

## MCP 1: Filesystem

**Status:** Available in Claude Code by default  
**Purpose:** Read and write your actual codebase

### What it enables

- Explore project structure without pasting files manually
- Read specific files or classes for context
- Write generated code directly into your project
- Search for patterns across the entire codebase

### Key prompts

```
Using the filesystem, read the directory structure of {{/path/to/project}}
and summarize the architecture and naming conventions.
```

```
Find all files that implement or extend {{ClassName}} and show me their structure.
```

```
Read {{path/to/file.php}} and explain what it does,
then propose improvements to make it more testable.
```

```
Search for all usages of {{methodName}} across the codebase
and list every place it's called with the calling context.
```

```
Read {{src/Service/}} and identify any classes that violate
the Single Responsibility Principle. List them with the reason.
```

### Integration with workflows

| Workflow step | Filesystem MCP use |
|---------------|--------------------|
| Explore codebase (Feature WF step 2) | Read project structure + relevant files |
| Implement (Feature WF step 4) | Write generated files directly |
| Debug: isolate layer (Debug WF step 3) | Read the suspected component |
| Code review | Read the file instead of pasting |

---

## MCP 2: Claude in Chrome (Browser)

**Status:** Available  
**Purpose:** Browse live documentation, inspect HTTP traffic, debug API responses

### What it enables

- Read live API documentation while implementing integrations
- Inspect actual HTTP requests/responses in DevTools
- Search for error messages and solutions in real time
- Verify deployed endpoints return correct responses
- Read GitHub issues and PRs without context switching

### Key prompts

```
Open {{docs URL}}, read the authentication section,
and summarize the steps needed for {{my stack}}.
```

```
Navigate to {{your local /api/endpoint}}, inspect the response,
and tell me if it matches the expected schema.
```

```
Search for "{{error message}}" and find the most relevant
GitHub issue or StackOverflow answer. Summarize the solution.
```

```
Open {{https://your-staging.domain/api/shipments}},
inspect the network response and headers, and tell me
if the Content-Type and auth headers are correct.
```

```
Go to {{https://docs.api.com/errors}} and list all
the error codes I need to handle for the {{/create}} endpoint.
```

### Integration with workflows

| Workflow step | Browser MCP use |
|---------------|-----------------|
| API Integration clarify | Read actual docs instead of pasting them |
| Debug: read the error | Search for the exact error message |
| Performance: profiling | Inspect DevTools waterfall |
| Architecture: research | Read comparison articles live |

---

## MCP 3: GitHub

**Status:** Connect in Claude.ai Settings → Integrations  
**URL:** `https://github.com/settings/tokens` (Personal Access Token)

### What it enables

- Search your org's repos for similar implementations before building
- Read issue/PR context before starting a task
- Find how a bug was introduced via git blame
- Check open PRs that might conflict with your work
- Create branches, commits, and PRs without leaving the conversation

### Key prompts

```
Search {{repo}} for examples of {{pattern or class name}}
and show the 3 most relevant results with file paths.
```

```
Read issue #{{N}} in {{owner/repo}} and summarize
what needs to be built, including any edge cases mentioned in comments.
```

```
Find all commits that touched {{filename}} in the last 30 days.
What changed and why?
```

```
List open PRs in {{repo}} that touch {{directory or service}}.
Are any of them likely to conflict with what I'm building?
```

```
Show me the git blame for {{filename}} lines {{from}}-{{to}}.
When was this logic introduced and why?
```

### Integration with workflows

| Workflow step | GitHub MCP use |
|---------------|----------------|
| Explore codebase | Search for existing patterns before building |
| Feature clarify | Read the issue before coding |
| Debug | Find commit that introduced the bug |
| Code review | Read open PRs to avoid conflicts |

---

## MCP 4: Database

**Status:** Install via your database MCP server (PostgreSQL, MySQL, etc.)  
**Setup:** See database-specific MCP server (e.g. `@modelcontextprotocol/server-postgres`)

### What it enables

- Inspect actual data rows when debugging data-related bugs
- Run EXPLAIN ANALYZE on slow queries for real profiling data
- Verify migrations applied correctly
- Understand data distribution before optimizing
- Test queries iteratively without switching to a DB client

### Key prompts

```
Run: EXPLAIN ANALYZE {{your slow query}}
Tell me where the bottleneck is and what index would fix it.
```

```
SELECT * FROM {{table}} WHERE {{condition}} LIMIT 5
Look at these rows and explain what they tell us about the bug.
```

```
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = '{{table}}'
Did the migration add the columns I expected?
```

```
SELECT status, COUNT(*) as cnt
FROM {{table}}
GROUP BY status ORDER BY 2 DESC

What does this distribution tell us?
Is anything unexpected here?
```

```
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
{{your query}}

Identify: seq scans on large tables, missing indexes,
high row estimate errors, and expensive sorts.
```

### Integration with workflows

| Workflow step | Database MCP use |
|---------------|------------------|
| Debug: isolate layer | Query for the unexpected data value |
| Performance: profiling | Run EXPLAIN ANALYZE without switching tools |
| Feature clarify | Check actual data shape before designing |
| Architecture | Understand current data volume and distribution |

---

## Recommended MCP Stack by Role

### Backend API developer

| MCP | Priority |
|-----|----------|
| Filesystem | Essential |
| GitHub | Essential |
| Database | Essential |
| Browser | High (API docs) |

### Platform / DevOps engineer

| MCP | Priority |
|-----|----------|
| Filesystem | Essential |
| GitHub | Essential |
| Browser | High (K8s docs, runbooks) |
| Database | Medium |

### Full-stack developer

| MCP | Priority |
|-----|----------|
| Filesystem | Essential |
| Browser | Essential (live app inspection) |
| GitHub | High |
| Database | Medium |

---

## Setting Up MCPs in Claude Code

### Filesystem (built-in)

Available by default. No setup needed.

### Browser (Claude in Chrome)

Install the Claude in Chrome extension from the Chrome Web Store, then it appears automatically in Claude Code.

### GitHub

1. Create a Personal Access Token at `github.com/settings/tokens`
2. Grant: `repo`, `read:org`, `read:user`
3. In Claude Code settings → MCP → Add server → GitHub
4. Paste token when prompted

### Database (PostgreSQL example)

Add to your `.mcp.json` or `~/.claude.json`:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://localhost/{{your_database}}"
      ]
    }
  }
}
```

---

## MCP Usage Patterns

### Pattern 1: Read before you answer

Before answering any question about your codebase, tell the AI to read the relevant files first:

```
Before answering, use the filesystem MCP to read:
- {{path/to/relevant/file}}
- {{path/to/related/test}}

Then answer: {{your question}}
```

### Pattern 2: Verify before you close

After implementing a feature, verify it was written correctly:

```
Use the filesystem to read the file you just created at {{path}}.
Confirm it matches the plan we agreed on and has no obvious issues.
```

### Pattern 3: Research before you design

Before proposing architecture, let the AI read the existing code:

```
Use the filesystem to read {{src/}} directory structure.
Identify the existing patterns for {{HTTP clients / repositories / services}}.
Then propose an architecture for {{new feature}} that fits these patterns.
```

### Pattern 4: Live docs during integration

While implementing an API integration:

```
Open {{https://docs.api.com/v2/endpoints/create-shipment}} using the browser MCP.
Read the request schema and all documented error codes.
Then implement the client using my existing pattern from {{ExistingClient.php}}.
```
