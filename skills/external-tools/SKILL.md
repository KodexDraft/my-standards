---
description: documents the CLIs installed in this environment and when to use them instead of their MCP equivalents; use before reaching for an MCP tool (Vercel, etc.) when the operation is already known
---

# External Tools (CLI vs MCP)

CLIs available in this environment, and when to use them instead of the matching MCP server. Add new CLIs at the bottom using the template.

## Rule of thumb

- If you already know the operation, use the CLI. One chained Bash call, no discovery steps (`which`, `--version`, `whoami`), output capped with `head` or `jq`.
- Use MCP when the CLI has no equivalent, when you need structured JSON to chain decisions, when the CLI is missing or not authenticated, or when the user asks for it.
- If the user says "use cli", use the CLI. If the CLI cannot do the job, say so and propose MCP instead of forcing it.
- Do not load a skill or MCP tool schemas just to answer a one-command question. The CLI is cheaper in tokens because there are no schemas and the output is a compact table.
- The CLI is only faster when the command is known. Guessing commands costs round trips, so use the tables below.

## Vercel CLI

- Binary: `vercel` (v64.2.0, `~/.nvm/versions/node/v24.13.1/bin/vercel`)
- Auth: logged in as `kodexdraft`, team `kodexdrafts-projects`
- Use it instead of the Vercel MCP whenever you already know the operation: deployment status, project management, environment variable configuration.

| Need | Command |
|---|---|
| Latest project and its status (one call) | `vercel project ls && vercel ls <project>` |
| Deployment status | `vercel ls [project]` |
| Only failed or ready deployments | `vercel ls <project> --status ERROR` (comma-separated: `--status BUILDING,READY`) |
| Production deployments only | `vercel ls <project> --environment production` |
| Deployments across all projects | `vercel ls --all` |
| Machine-readable output | `vercel ls <project> --json --limit 5` |
| Deployment details (build time, aliases, machine) | `vercel inspect <deployment-url>` |
| List env vars (names and environments) | `vercel env ls [environment]` |
| Add, update, remove an env var | `vercel env add\|update\|remove <name> [environment]` |
| Pull env vars to a local file | `vercel env pull [file]` (default `.env.local`) |
| Run a command with the project's env | `vercel env run -- <command>` |
| Project management | `vercel project ls`, `vercel project inspect [name]`, `vercel project create <name>` |
| Runtime logs | `vercel logs <deployment-url>` |
| Deploy, promote, roll back | `vercel deploy`, `vercel promote`, `vercel rollback` |
| Link a folder to a project | `vercel link` |

Notes:

- Never print env var values. `vercel env ls` shows names only; keep it that way.
- Commands that need a project work without arguments only in a linked folder (`.vercel/project.json`). The `my-standards` folder is not linked, so pass the project name or use `--cwd <linked-folder>`.
- Outward-facing or hard-to-reverse operations (production deploy, `env add|update|remove`, `promote`, `rollback`) need the user's confirmation first.
- Prefer MCP for what the CLI does not cover well: runtime errors and observability queries, firewall, drains, feature flags, Edge Config, Sandbox, Toolbar comments, and documentation search.
- For a one-off status question, skip the `vercel:status` skill. It is a large load and pushes toward MCP.

## Installed, not yet documented

Found with `command -v` on 2026-10-15. Add a section for each when you start using it.

- `gh` (`/opt/homebrew/bin/gh`): GitHub
- `gcloud` (`/opt/homebrew/bin/gcloud`): Google Cloud
- `jq` (`/usr/bin/jq`): JSON filtering, useful with any `--json` flag
- `git`, `npm`, `node`

Checked and not installed: `supabase`, `convex`, `neonctl`, `stripe`, `aws`, `docker`.

## Suggestions

- Chain independent commands in a single Bash call (`a && b`) instead of one call per command.
- Use `--json | jq` only when the output has to be parsed by a script. The default table is cheaper to read.
- Cap output with `--limit` or `head`. The less text that comes back, the fewer tokens it costs.
- To cut latency, tell the agent: "cli, single call, no pre-checks".
- For each CLI, note the auth check command, one common chained command, and what the CLI cannot do. That is what lets the agent skip exploration.
- Keep each CLI to one table and one "use MCP when" line. This file loads into context in full, so it should stay short.

## Template for a new CLI

Copy this block and fill it in.

```markdown
## <Name> CLI

- Binary: `<command>` (<version>, <path>)
- Auth: <how it is logged in, or the command that checks it>
- Use it instead of the <Name> MCP when: <operations>

| Need | Command |
|---|---|
| <most common task, one chained call> | `<command>` |
| ... | ... |

Notes:

- Confirm with the user before: <hard-to-reverse operations>
- Prefer MCP for: <what the CLI cannot do>
```
