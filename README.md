# user-flow-dev

A skill for user-flow-centered development without creating a second backlog.

User flows are durable behavioral contracts for critical journeys. They describe actor-visible paths, branches, failures, and acceptance criteria that future agents must understand before changing behavior. GitHub Issues, Projects, or another configured tracker remain the sole owners of implementation scope, dependencies, priority, and status.

## Commands

| Command | Purpose |
|---|---|
| `/user-flow-dev init` | Propose and create 4–8 initial critical flows. |
| `/user-flow-dev add <description>` | Add one critical flow after clarification and approval. |
| `/user-flow-dev report <description>` | Reconcile reported behavior with a flow and external tracking. |
| `/user-flow-dev decision <description>` | Record an approved behavioral change. |
| `/user-flow-dev check [domain or FLOW-ID]` | Read-only verification against code and tests. |

## Registry

By default, initialization creates:

```text
docs/user-flows/
  overview.md
  flows/
    UF-<DOMAIN>-NNN.md
```

Each flow lives in one stable file. There are no task files, implementation statuses, todo lists, or archives.

## Tracker integration

Issues and pull requests reference flow IDs without copying behavioral acceptance criteria. Pull-request evidence uses stable keys such as `UF-ACCESS-006/AC2`. Before implementation, each affected flow is pinned to its exact Git commit:

```markdown
## User-flow contracts
- `UF-ACCESS-006` — [`abc1234`](https://github.com/OWNER/REPO/blob/FULL_SHA/docs/user-flows/flows/UF-ACCESS-006.md)
```

If the current file differs from the pinned contract, work pauses for scope reconciliation. Git history preserves the old contract; the issue tracker preserves implementation history.

## Install

```bash
git clone https://github.com/erzats/user-flow-dev.git ~/.agents/skills/user-flow-dev
```

Use another runtime-specific personal skills directory when required.
