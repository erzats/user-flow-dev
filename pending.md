# `/user-flow-dev pending`

List flows that have no associated task. **Discovery only — generates nothing.**

## What this command does and does not do

- ✅ Reads `overview.md`, `tasks/todo.md`, and `tasks/archive/`
- ✅ Identifies flows whose IDs do not appear in any task file
- ✅ Prints a grouped, scannable list
- ❌ Does not create tasks. To create one, the user runs `/user-flow-dev task <FLOW-ID>`.
- ❌ Does not propose new flows. To add one, the user runs `/user-flow-dev add ...`.
- ❌ Does not modify any files.

If you find yourself reaching for Write or Edit, you misunderstood — this is read-only.

## Procedure

### Step 1 — Read

- `.claude/user-flows/overview.md` (canonical flow list)
- `.claude/user-flows/tasks/todo.md` (active tasks)
- `.claude/user-flows/tasks/archive/*.md` (completed tasks — flows tied to a done task are still "covered")

If any of these files don't exist, tell the user that user-flows aren't initialized yet and stop. Do not bootstrap.

### Step 2 — Compute the set

A flow is "pending" if its `UF-...` ID does **not** appear in:
- Any line of `todo.md`, AND
- Any file in `tasks/archive/`

Treat archived tasks as "covered" — that flow has been worked on; not pending.

Filter out flows whose status (in their domain file) is `superseded` or `deferred`. They are intentionally not actionable.

### Step 3 — Group and print

Group by domain. Within a domain, sort by ID. Format:

```
Pending flows (no task):

auth/
  UF-AUTH-003 | First-time account provisioning | tags: signup, sso
  UF-AUTH-005 | Session expiry / re-auth | tags: session, refresh

payments/
  UF-PAYMENTS-004 | Subscription renewal succeeds | tags: stripe, renewal

(3 of 24 flows have no associated task)
```

If every flow has a task, print:

```
No pending flows. All 24 documented flows have at least one task (active or archived).
```

If a domain has no pending flows, omit it entirely — don't print empty headers.

If there are deferred or superseded flows, optionally print a separate "Deferred" section so the user remembers they exist:

```
Deferred (not actionable):
  UF-NOTIF-005 | Member manages notification preferences | tags: settings
```

Skip the section if it would be empty.

## Anti-patterns specific to pending

- Suggesting which flows the user should create tasks for. The list itself is the suggestion.
- Auto-generating tasks for the user. They use `task` for that.
- Including superseded/deferred flows in the main pending list.
- Re-reading domain files for content that's already in the overview. The overview is the index for a reason — only crack open a domain file if you genuinely need a flow's status.
