---
name: user-flow-dev
description: Use when the user types /user-flow-dev (any subcommand), when the project contains .claude/user-flows/, when work is about to touch a documented flow's domain, when a UF-* flow ID appears in conversation, or when finishing a task tied to a flow. Triggers also include: starting on a feature in a project with documented flows; when a proposed change risks violating an existing flow's acceptance criteria; and when the user reports a bug, regression, or "should work like X" expectation that maps to user-visible behavior — propose `/user-flow-dev report` to capture it as a flow change or new flow.
---

# user-flow-dev

A registry of **user flows** — behavioral descriptions of how the product should work — that lives at `.claude/user-flows/` and is consulted by Claude before, during, and after work. Flows survive session resets so new requirements don't quietly break old behavior. See `README.md` for the rationale.

## Routing

The first word of `$ARGUMENTS` selects the subcommand. Read the matching reference file in full before doing anything.

| Subcommand | Reference file | What it does |
|---|---|---|
| `init` | `init.md` | First-time scan: infer domains, propose flows, write files. |
| `add` | `add.md` | **Conversational.** Add one flow. Never one-shot. |
| `pending [status]` | `pending.md` | List flows that need attention; optional status slug narrows to one status. Read-only. |
| `task <FLOW-ID>` | `task.md` | Generate a single task file for one flow. |
| `report <description>` | `report.md` | **Conversational.** Route a user-reported issue to an existing flow (modify) or a new flow (create), then generate a task. |
| `check [domain]` | `check.md` | Verify acceptance criteria still hold against current code. |
| `done <TASK-ID>` | `done.md` | Stage a task as `needs manual validation` after implementation. Does not archive. |
| `validated <TASK-ID>` | `validated.md` | Archive a task after the user has manually validated it. |
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

## Status lifecycle

Every flow has a `Status:` field. `pending` and `check` use it to decide what to surface.

| Status | Set by | Surfaces in `pending`? | Meaning |
|---|---|---|---|
| `init` | `init` | No | Inferred from existing code; not yet evaluated by `check`. |
| `not started` | `add`, human | Yes | Intended behavior; implementation hasn't begun. |
| `incomplete` | `check` | Yes | Some ACs verified, some not implemented. |
| `issues` | `check` | Yes | At least one AC `✗ broken`. |
| `needs manual validation` | `check`, `done` | Yes | At least one AC `⚠ unclear` (and none broken), or implementation just landed and awaits human verification. |
| `completed` | `check`, `validated` | No | All ACs verified holding. |
| `deferred` | human only | No | Future behavior, not currently planned. `check` will not overwrite. |
| `superseded by UF-X-NNN` | human only | No | Replaced by another flow. `check` will not overwrite. |

The verdict precedence `check` uses to compute these statuses (broken > unclear > incomplete > completed) is in `check.md` Step 4. Don't restate it elsewhere.

## Tasks and findings

There is at most one open task per flow. Tasks live in `tasks/<domain>/TASK-NNN.md` and use a global TASK-NNN sequence. The header field `**Source:** manual` or `**Source:** check` records who created the task.

When a task exists for a flow, the flow's detail section in the domain file carries an `Active task: tasks/<domain>/TASK-NNN.md` line directly under `Status:`. This is added by `task` or `check` when the task is created and removed by `done` or `check` when the task is archived — keeping each flow's detail honest about whether work is actively tracked.

Inside any task, the `## Findings` section is owned by `check`. It is added on the first check run that surfaces issues for the linked flow and refreshed on every subsequent run. Every other section in the task — `## Scope`, `## Notes`, custom sections — belongs to the human or the command that created the task; `check` does not touch them.

## CLAUDE.md integration

After `init`, the project's CLAUDE.md gets a usage block telling future Claude when to read flow files, when to call `done`/`validated`/`check`, and to flag changes that would violate ACs. The exact snippet and placement rules are in `init.md` Step 9. The update is always preview-then-confirm.

## Anti-patterns — stop yourself if you catch yourself doing these

(File-scope and "infer domains" rules are covered in the Four invariants above; this list is for skill-wide behaviors not already pinned there.)

- Adding a flow without a clarification round. See `add.md`.
- Generating tasks in bulk. `task` operates on one flow at a time.
- Marking a flow's acceptance criteria as verified without actually checking them against code or tests. See `check.md`.
- Deleting completed task files. They go to `tasks/archive/`.
