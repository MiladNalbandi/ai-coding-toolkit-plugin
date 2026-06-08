---
name: mcp-toolkit
description: >
  How to set up and use MCP servers for coding — filesystem, browser, GitHub, and
  database — with key prompts and workflow integration for each. Use when
  wiring up MCP tools or deciding which MCP to use for a coding task. Trigger on
  "set up MCP", "use the filesystem or github or database tool", "which MCP should I use".
compatibility: opencode
---

# MCP Toolkit — AI Coding with Real Tools

> **MCP task:** $ARGUMENTS


> MCPs (Model Context Protocol servers) let your AI coding assistant read files, browse docs, query databases, and interact with GitHub — without you pasting anything manually.

> **opencode note:** opencode configures MCP servers in `opencode.json` (project root) or
> `~/.config/opencode/opencode.json` (global), under an `mcp` key. Each server is either a
> **local** server (`"type": "local"` with a `command` array) or a **remote** server
> (`"type": "remote"` with a `url`). Add `"enabled": true` to activate it. Throughout this
> skill, wherever a server needs configuring, drop it into that `mcp` block — see the
> setup section below for concrete examples.

---

## Why MCPs change how you code with AI

Without MCPs, you paste code → AI reads it → AI responds → you manually apply changes.

With MCPs, AI reads your actual files, browses live docs, runs queries against your DB, and writes back to your filesystem. The feedback loop shrinks from minutes to seconds.

---

## MCP 1: Filesystem

**Status:** Available via the `@modelcontextprotocol/server-filesystem` MCP server (add to `opencode.json`'s `mcp` block). opencode also has built-in file read/write/grep tools that cover most of this natively.
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

## MCP 2: Browser

**Status:** Available via a browser MCP server (e.g. a Playwright/Puppeteer-based MCP, or a Chrome-extension-backed browser MCP). Add it to `opencode.json`'s `mcp` block — as a local server pointing at the browser MCP command, or a remote server if it exposes an HTTP endpoint.
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

**Status:** Add the GitHub MCP server to `opencode.json`'s `mcp` block — typically as a **remote** server (`"type": "remote"`) pointing at GitHub's hosted MCP endpoint, or as a **local** server running `@modelcontextprotocol/server-github`.
**Auth:** Personal Access Token from `https://github.com/settings/tokens`, passed via an `environment` entry on the server (or an auth header for the remote variant).

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

**Status:** Install via your database MCP server (PostgreSQL, MySQL, etc.) and register it in `opencode.json`'s `mcp` block as a **local** server.
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

## Setting Up MCPs in opencode

opencode reads MCP server definitions from `opencode.json` (project root, checked into the
repo) or `~/.config/opencode/opencode.json` (global). All servers live under a top-level
`mcp` key. Two server types:

- **Local** (`"type": "local"`): opencode spawns a process. Give it a `command` array
  (program + args) and optional `environment` map for secrets like tokens.
- **Remote** (`"type": "remote"`): opencode connects to an HTTP MCP endpoint. Give it a
  `url` and optional `headers` for auth.

Add `"enabled": true` to each server to turn it on.

### Filesystem

Either rely on opencode's built-in file tools, or register the filesystem MCP server:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "filesystem": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"],
      "enabled": true
    }
  }
}
```

### Browser

Install your chosen browser MCP server (a Playwright/Puppeteer-based one, or a
Chrome-extension-backed browser MCP), then register it. For a locally spawned server:

```json
{
  "mcp": {
    "browser": {
      "type": "local",
      "command": ["npx", "-y", "<your-browser-mcp-package>"],
      "enabled": true
    }
  }
}
```

If your browser MCP runs as a remote endpoint instead, use `"type": "remote"` with its `url`.

### GitHub

1. Create a Personal Access Token at `github.com/settings/tokens`
2. Grant: `repo`, `read:org`, `read:user`
3. Register the GitHub MCP server in `opencode.json`. Local variant:

```json
{
  "mcp": {
    "github": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-github"],
      "environment": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "{{your_token}}"
      },
      "enabled": true
    }
  }
}
```

For GitHub's hosted MCP server, use a remote entry instead:

```json
{
  "mcp": {
    "github": {
      "type": "remote",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": { "Authorization": "Bearer {{your_token}}" },
      "enabled": true
    }
  }
}
```

### Database (PostgreSQL example)

```json
{
  "mcp": {
    "postgres": {
      "type": "local",
      "command": [
        "npx",
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://localhost/{{your_database}}"
      ],
      "enabled": true
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
