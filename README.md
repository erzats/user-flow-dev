# user-flow-dev

A Claude Code skill that maintains a living registry of **user flows** — behavioral descriptions of how a product should work — under `.claude/user-flows/` in your project. Designed for long Claude Code sessions where context drifts and old flows quietly break.

Deliberately **not** called "user stories" — these are behavioral specs with branches and acceptance criteria, not backlog items.

## What it does

- **Infers domains** from your codebase (auth, payments, plus app-specific ones like `hunt-builder`) — you never name them.
- **Generates a dense, scannable index** so an agent can find relevant flows without reading every domain file.
- **Forces a clarification round** before adding a flow, so branches and edges actually get captured.
- **Verifies acceptance criteria against real code** with a strict no-vibes-based-verification rule.
- **Tracks tasks per flow** with a two-phase close: `done` stages a task as `needs manual validation`, `validated` archives it once the user has actually exercised the flow.

## Subcommands

| Command | What it does |
|---|---|
| `/user-flow-dev init` | Scan codebase, infer domains, propose flows, write files. |
| `/user-flow-dev add <description>` | Conversational add of a single flow with required clarification round. |
| `/user-flow-dev report <description>` | Conversational route of a user-reported issue to an existing flow (modify) or a new flow (create), then generate a task. |
| `/user-flow-dev pending [status]` | List flows that need attention; optional status slug (e.g. `issues`, `needs-manual-validation`) narrows to one status. Read-only. |
| `/user-flow-dev task <FLOW-ID>` | Generate a task file for one flow. |
| `/user-flow-dev check [domain]` | Verify acceptance criteria still hold against current code; reconcile statuses and tasks. |
| `/user-flow-dev done <TASK-ID>` | Stage a task as `needs manual validation` after implementation. Does not archive. |
| `/user-flow-dev validated <TASK-ID>` | Archive a task after the user has manually validated it; flips the linked flow to `completed`. |

## Install (personal, all projects)

Clone into your personal skills directory:

```bash
git clone https://github.com/erzats/user-flow-dev.git ~/.claude/skills/user-flow-dev
```

On Windows (PowerShell):

```powershell
git clone https://github.com/erzats/user-flow-dev.git $HOME\.claude\skills\user-flow-dev
```

The `/user-flow-dev` slash command appears the next time you start Claude Code.

## Install (project-local, just this repo)

```bash
git clone https://github.com/erzats/user-flow-dev.git .claude/skills/user-flow-dev
```

Commit `.claude/skills/user-flow-dev/` if you want the team to share it; otherwise add it to `.gitignore`.

## Update

```bash
cd ~/.claude/skills/user-flow-dev && git pull
```

## File structure produced

```
.claude/user-flows/
  overview.md          ← one-line-per-flow index
  domains/
    <domain>.md        ← per-domain summary, flow index, full flow detail
  tasks/
    todo.md            ← one-line-per-task index
    archive/           ← completed task files (kept, never deleted)
    <domain>/          ← active task files
```

After init, the skill offers to add a small instruction block to your project's CLAUDE.md so future Claude sessions read `overview.md` on session start, call `/user-flow-dev done <TASK-ID>` when finishing implementation work (which stages the task as `needs manual validation`), and leave `/user-flow-dev validated <TASK-ID>` to the user once they've manually confirmed the flow works.

## Design constraints

- **Token efficiency.** Indices are dense pipe-delimited lines (`UF-AUTH-001 | Sign in with SSO | auth | tags: sso, session`). An agent reads `overview.md` first to decide which domain file to load.
- **Scope safety.** All file writes are confined to `.claude/user-flows/`. The CLAUDE.md update during `init` is preview-then-confirm, never silent.
- **No content redundancy.** The index line is duplicated (overview + domain index) for scannability; the content (branches, AC) lives in exactly one place.
- **Behavioral, not implementation.** Flow detail describes what the user experiences — not tables, RLS, endpoints, or library calls.

## License

MIT
