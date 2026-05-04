# `/user-flow-dev pending`

List flows that need attention based on their `Status:` field. **Discovery only — generates nothing.**

## What this command does and does not do

- ✅ Reads `overview.md` and each domain file under `domains/`
- ✅ Identifies flows whose status is one of `not started`, `incomplete`, `issues`, `needs manual validation`
- ✅ Prints a grouped, scannable list
- ❌ Does not create tasks. To create one, the user runs `/user-flow-dev task <FLOW-ID>`.
- ❌ Does not propose new flows. To add one, the user runs `/user-flow-dev add ...`.
- ❌ Does not modify any files.

If you find yourself reaching for Write or Edit, you misunderstood — this is read-only.

## Procedure

### Step 1 — Read

- `.claude/user-flows/overview.md` (canonical flow list)
- `.claude/user-flows/domains/*.md` (to read each flow's `Status:` field)

If `overview.md` does not exist, tell the user that user-flows aren't initialized yet and stop. Do not bootstrap.

### Step 2 — Compute the set

A flow is "pending" if its `Status:` is **one of**:
- `not started`
- `incomplete`
- `issues`
- `needs manual validation`

A flow is **not** pending if its status is:
- `init` — inferred from code, not yet evaluated by `check`. Run `check` if you want a verdict.
- `completed` — verified working at last `check`.
- `deferred` — intentionally not built.
- `superseded by UF-X-NNN` — replaced.

Status is the only signal `pending` cares about. Task association (presence in `todo.md` or `archive/`) is **not** considered — that is the job of `/user-flow-dev task list` (if any), not `pending`.

### Step 3 — Group and print

Group by domain. Within a domain, sort by ID. Annotate each line with the flow's status so the user can prioritize. Format:

```
Pending flows (need attention):

auth/
  UF-AUTH-003 | not started           | First-time account provisioning | tags: signup, sso
  UF-AUTH-005 | issues                | Session expiry / re-auth | tags: session, refresh

payments/
  UF-PAYMENTS-004 | needs manual validation | Subscription renewal succeeds | tags: stripe, renewal

(3 of 24 flows need attention; 18 completed, 2 init, 1 deferred)
```

If no flows are pending, print:

```
No pending flows. All 24 flows are either `completed`, `init`, `deferred`, or `superseded`.
```

If everything is still `init` (no `check` has run yet), say so explicitly so the user understands `pending` will stay empty until they run `check`:

```
No pending flows — all 24 flows are still in `init` status (inferred from code, not yet evaluated).
Run `/user-flow-dev check` to evaluate ACs against current code; flows that fail will surface here.
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
- Including `init`, `completed`, `deferred`, or `superseded` flows in the main pending list.
- Filtering by task association instead of by status. Tasks are orthogonal to pending in the new model.
- Re-reading domain files for AC content. Read them only to extract the `Status:` line for each flow.
