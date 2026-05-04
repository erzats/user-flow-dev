---
name: user-flow-dev
description: Use when the user types /user-flow-dev (any subcommand), when the project contains .claude/user-flows/, when work is about to touch a documented flow's domain, when a UF-* flow ID appears in conversation, or when finishing a task tied to a flow. Triggers also include: starting on a feature in a project with documented flows, or when a proposed change risks violating an existing flow's acceptance criteria.
---

# user-flow-dev

A registry of **user flows** — behavioral descriptions of how the product should work — that lives at `.claude/user-flows/` and is consulted by Claude before, during, and after work.

The problem this solves: in long sessions, context drifts. New requirements quietly break old flows and nobody notices until production. This skill keeps intended behavior as a first-class artifact that survives session resets and forces a verification loop.

## Why "flows" not "stories"

Flows are *behaviors*, not backlog items. They have branches, error paths, acceptance criteria, and stable IDs. They survive sprints. "Regression" means a documented flow's acceptance criteria stopped holding.

## Routing

The first word of `$ARGUMENTS` selects the subcommand. Read the matching reference file in full before doing anything.

| Subcommand | Reference file | What it does |
|---|---|---|
| `init` | `init.md` | First-time scan: infer domains, propose flows, write files. |
| `add` | `add.md` | **Conversational.** Add one flow. Never one-shot. |
| `pending` | `pending.md` | List flows with no associated task. Read-only. |
| `task <FLOW-ID>` | `task.md` | Generate a single task file for one flow. |
| `check [domain]` | `check.md` | Verify acceptance criteria still hold against current code. |
| `done <TASK-ID>` | `done.md` | Archive a completed task. |
| (none / unknown) | — | Print this table and stop. |

The canonical shape of a single flow's detail section is in `flow-template.md`. `init` and `add` both reference it.

## Four invariants — read before any subcommand

These hold across every subcommand. The reference files assume you know them.

### 1. File structure (fixed — do not propose alternatives)

```
.claude/user-flows/
  overview.md          ← one-line-per-flow index, domain list, nothing else
  domains/
    <domain>.md        ← 2-sentence domain summary + flow index + flow detail
  tasks/
    todo.md            ← one-line-per-task index with status
    archive/           ← completed task files (never deleted)
    <domain>/          ← TASK-NNN.md files for in-progress tasks
```

If any of this is missing when you need it, **create it without asking — but only inside `.claude/user-flows/`.**

Hard scope rules:
- Do not migrate, modify, or delete files outside `.claude/user-flows/`.
- Do not silently rewrite existing project documentation in `docs/`, `README.md`, or anywhere else.
- The one exception is the CLAUDE.md update during `init`, which is always preview-then-confirm — never silent.
- When you create files inside `.claude/user-flows/`, list every file path you wrote in your final confirmation message.

Do not ask the user where flows should live or whether to use a different folder. The structure above is the answer.

### 2. Behavioral, not implementation

A flow describes what the user experiences and what branches exist. It does not describe tables, RLS, endpoints, SDK methods, or component names. Acceptance criteria are observable from outside the system — "subscription state converges within 5 minutes" not "webhook handler updates the row".

### 3. Indices are dense — pipe-delimited, one line per flow

`overview.md` and the index section at the top of each domain file use this exact format:

```
UF-<DOMAIN>-NNN | <Title> | <domain> | tags: <tag>, <tag>, <tag>
```

`todo.md` uses the same shape for tasks:

```
TASK-NNN | <one-line summary> | <domain> | <FLOW-ID> | <status>
```

`<status>` for tasks is `pending`, `in-progress`, or `done`.

If you find yourself writing a paragraph in any index, stop and rewrite as one line. Em-dashes, narrative, branch counts — none of that goes in the index. Detail lives in the per-flow section of the domain file.

### 4. No content redundancy across files

- `overview.md` lists flows but never describes them. The index *line* is duplicated between `overview.md` and the domain's index section; the *content* (branches, AC, preconditions) is not.
- Task files reference `UF-...` IDs. They never restate flow content.

## Domain inference (always automatic)

Domain names are inferred from the codebase, never asked. See `init.md` for the inference procedure. New flows added later go through the same inference (see `add.md`). Generic domains (`auth`, `payments`, `onboarding`, `notifications`) are normal; application-specific domains (`hunt-builder`, `admin-dashboard`, `mobile-reader`) are encouraged when they earn their keep.

**Default rule: a domain needs at least 3 flows.** Prefer merging domains with fewer than 3 flows into the closest neighbor.

**Exception:** keep a small domain when it has *distinct actors, distinct lifecycle, or distinct risk profile* from its neighbors. Two payment-fraud flows are not a thin domain — they are a real one with elevated risk; keep them separate even if there are only two. Two profile-edit flows that overlap with a `settings` domain are a thin domain; merge them.

When in doubt, prefer the hard rule (merge). It is easier to split a fat domain later than to chase down flows scattered across thin ones.

## Flow IDs

`UF-<DOMAIN>-NNN` where `<DOMAIN>` is the uppercased domain name (or short prefix for long names — `PAYMENTS` not `BILLING-AND-SUBSCRIPTIONS`). Numbers start at 001, append-only. Never renumber. If a flow is superseded, mark its detail section `Status: superseded by UF-X-NNN`; do not delete or renumber.

## Status values

Every flow has a `Status:` field. Status reflects the flow's lifecycle position; `pending` and `check` use it to decide what to surface.

| Status | Set by | `pending` surfaces? | `check` examines? | Meaning |
|---|---|---|---|---|
| `init` | `init` (default) | No | Yes | Inferred from existing code; not yet evaluated by `check`. The sentinel for "we wrote down what the code seems to do, but haven't verified ACs against it." |
| `not started` | `add` (default), human | Yes | Yes | Intended behavior; implementation hasn't begun. |
| `incomplete` | `check` | Yes | Yes | Some ACs verified (`✓ holds`), some have no implementation (`– not implemented`). |
| `issues` | `check` | Yes | Yes | At least one AC `✗ broken`. Highest-priority status when multiple verdicts apply. |
| `needs manual validation` | `check` | Yes | Yes | At least one AC `⚠ unclear` (and none broken). The flow needs a human to confirm. |
| `completed` | `check` | No | Yes | All ACs `✓ holds`. The flow is verified working as of the last `check`. |
| `deferred` | human | No | No | Future behavior, not currently planned. `check` cannot infer this — only humans set it. |
| `superseded by UF-X-NNN` | human | No | No | Replaced by another flow. Kept for traceability. `check` cannot infer this — only humans set it. |

### Who writes what

- **`init`** writes `Status: init` for every flow it generates. After `init`, `pending` is empty.
- **`add`** writes `Status: not started` for every new flow (since `add` typically describes new or about-to-build behavior). The flow surfaces in `pending` immediately.
- **`check`** writes one of `incomplete` / `issues` / `needs manual validation` / `completed` based on the per-flow verdict (see `check.md` "Status mapping"). `check` never overwrites `deferred` or `superseded` — those are intent flags it has no signal for.
- **Humans** set `deferred` (intentional non-implementation) and `superseded by UF-X-NNN` (replacement relationship). Both encode product/refactoring intent that cannot be derived from code analysis.

### Verdict precedence in `check`

When a flow has mixed AC verdicts, `check` picks the highest-priority status that applies:

1. Any `✗ broken` → `issues`
2. Any `⚠ unclear` (and none broken) → `needs manual validation`
3. Mix of `✓ holds` and `– not implemented` → `incomplete`
4. All `– not implemented` → `not started`
5. All `✓ holds` → `completed`

## CLAUDE.md integration

After `init`, the project's CLAUDE.md gets a small instruction block (full snippet in `init.md`) telling future Claude to: read `overview.md` at session start, load the relevant domain file before touching its area, call `/user-flow-dev done <TASK-ID>` automatically when finishing implementation work, and flag any change that would violate a flow's acceptance criteria *before* making it. The CLAUDE.md update is always preview-then-confirm.

## Anti-patterns — stop yourself if you catch yourself doing these

- Asking the user "what domains do you want?" — infer them.
- Asking "where should flows live?" or "should I use a different folder?" — the structure is fixed.
- Writing files outside `.claude/user-flows/` (except the previewed CLAUDE.md update during `init`).
- Writing prose in the index. One pipe-delimited line per flow.
- Including schema, SQL, RLS policies, endpoint paths, or library names in flow detail. That is implementation.
- Adding a flow without a clarification round. See `add.md`.
- Generating tasks in bulk. `task` operates on one flow at a time.
- Marking a flow's acceptance criteria as verified without actually checking them against code or tests. See `check.md`.
- Deleting completed task files. They go to `tasks/archive/`.
- Creating a new domain for a single flow. Fold thin domains.
