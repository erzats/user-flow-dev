# user-flow-dev

A Claude Code skill that gives long-running coding agents **durable memory of how your product is supposed to behave**. Lives at `.claude/user-flows/` in your repo, domain-organized, indexed so an agent can find the relevant slice without reading everything.

## The problem it solves

In long Claude Code projects, agents lose track of how previously-built pieces were supposed to work. You implement feature A in one session; ten sessions later you ask for feature B, which is tangential to A — and the agent quietly breaks A because nothing in its context says how A was supposed to behave. You squash, revert, re-explain, repeat.

External trackers (Linear, Jira, Notion) don't fix this — they live outside the repo and can't feed back into agent context. CLAUDE.md doesn't shard, so it gets too big to be useful. Cucumber feature files are test-runner-shaped, not registry-shaped.

## The loop

- **Capture** — `init` scans the codebase (or a project description, on a new project), infers domains, and writes one behavioral flow per piece of user-visible functionality. You never name the domains.
- **Consult** — future sessions read `overview.md` first; the dense pipe-delimited index lets the agent pick the right domain file in a single read, keeping token cost flat as the project grows.
- **Correct** — when the agent gets behavior wrong, `report` routes your natural-language correction into either a flow edit (capturing the nuance) or a new flow, plus a task to fix the code. The next session inherits the correction instead of you re-explaining.
- **Verify** — `check` reconciles flow acceptance criteria against the actual code, with a strict no-vibes-based-verification rule.
- **Close** — `done` stages a task as `needs manual validation`; `validated` archives it once you've actually exercised the flow.

## Use cases

- **Existing project (primary).** `init` mines the code and docs to surface flows you've already built, so the registry starts already aligned with reality. From there, `add`/`report` extend it as the product evolves.
- **Greenfield project (experimental — please report issues).** `init` accepts a project description, asks a short clarification round about target users, key features, and scope, then proposes domains and flows from the description alone. You get a behavioral skeleton to work against from the first commit. Domain split/merge isn't supported yet, so prefer broader, more general domains at the start — they can be reorganized manually as the project grows.

## Works in natural language, anywhere

Both `add` and `report` are conversational. You can be at the keyboard, or in the Claude Code mobile/web app — on a walk, in a car, away from a screen — and dump:

- "I just thought of an edge case for signup: what if the user's SSO provider returns an email that conflicts with an existing local account?"
- "I noticed during testing that admin role changes don't propagate to active sessions until logout."
- "What if lesser admins could approve refunds under $50?"

The skill triages each one into a flow edit, a new flow, or a new task — whichever fits. By the time you're back at the keyboard, `pending` shows a queue ready to work on. You implement it on your own terms; the skill doesn't tell you how.

## What it deliberately does not do

- **Not a user-story tool.** These are behavioral specs with branches and acceptance criteria, not backlog items.
- **Not an implementation guide.** No tables, RLS, endpoints, component names, or "how to build it" advice ever appears in a flow. The skill records *what* the application should do — *how* to build it is the user's domain. There is no `implement` subcommand and there will not be one.
- **Not a sprint board.** No estimates, no assignees, no sprints. Tasks are just "the next work tied to this flow."

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
