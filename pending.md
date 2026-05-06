# `/user-flow-dev pending [status]`

List flows that need attention based on their `Status:` field. **Discovery only — generates nothing.**

With no argument, surfaces flows that need work (the default attention set). With an explicit status slug, lists every flow that matches that status — useful for narrowing to just `issues`, or for surfacing `init` / `completed` / `deferred` flows that the default view hides.

## What this command does and does not do

- ✅ Reads `overview.md` and each domain file under `domains/`
- ✅ With no arg: identifies flows whose status is one of `not started`, `incomplete`, `issues`, `needs manual validation`
- ✅ With a status slug: identifies flows whose status matches that slug
- ✅ Prints a grouped, scannable list
- ❌ Does not create tasks. To create one, the user runs `/user-flow-dev task <FLOW-ID>`.
- ❌ Does not propose new flows. To add one, the user runs `/user-flow-dev add ...`.
- ❌ Does not modify any files.

If you find yourself reaching for Write or Edit, you misunderstood — this is read-only.

## Status slugs

The optional argument is a slug — multi-word statuses use hyphens. One slug per invocation; this command does not support comma-separated multiples or repeated args.

| Slug | Matches `Status:` value | Surfaces in default view? |
|---|---|---|
| `not-started` | `not started` | Yes |
| `incomplete` | `incomplete` | Yes |
| `issues` | `issues` | Yes |
| `needs-manual-validation` | `needs manual validation` | Yes |
| `init` | `init` | No |
| `completed` | `completed` | No |
| `deferred` | `deferred` | No |
| `superseded` | any value starting with `superseded by ` | No |

Anything else is invalid — see Step 1.

## Procedure

### Step 1 — Validate the input

`$ARGUMENTS` after `pending` is either empty or one slug from the table above.

- Empty → use the default attention set (Step 2a).
- Recognized slug → filter to that status (Step 2b).
- Multiple args → "One status per `pending` command. Run it twice if you need two views." and stop.
- Unrecognized slug → print the slug table from this file and stop. Do not guess; do not fall back to the default view.

### Step 2a — Compute the default set (no argument)

A flow is "pending" by default if its `Status:` is **one of**:
- `not started`
- `incomplete`
- `issues`
- `needs manual validation`

A flow is **not** in the default set if its status is `init`, `completed`, `deferred`, or `superseded by ...`. Use the explicit slug to surface those.

### Step 2b — Compute the filtered set (slug given)

Match every flow whose `Status:` equals the slug's mapped value (per the table). For `superseded`, match any flow whose status string starts with `superseded by ` (case-sensitive — that's how `add` and humans write it).

Status is the only signal `pending` cares about. Task association (presence in `todo.md` or `archive/`) is not considered — that is the job of `/user-flow-dev task list` (if any), not `pending`.

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

When a slug was given, change the heading to name the slug and drop the parenthetical breakdown (it's redundant — every line has the same status):

```
Flows with status `issues`:

auth/
  UF-AUTH-005 | Session expiry / re-auth | tags: session, refresh

payments/
  UF-PAYMENTS-002 | Subscribe via Stripe Checkout | tags: stripe, subscription
  UF-PAYMENTS-007 | Refund flow | tags: stripe, refund

(3 of 24 flows match)
```

If no flows match (default view): print the standard "No pending flows" message. If `init` is the dominant status, name it explicitly so the user knows to run `check`:

```
No pending flows — all 24 flows are still in `init` status (inferred from code, not yet evaluated).
Run `/user-flow-dev check` to evaluate ACs against current code; flows that fail will surface here.
```

If no flows match (slug given):

```
No flows with status `issues` (of 24 total).
```

If a domain has no matching flows, omit it entirely — don't print empty headers.

If there are deferred or superseded flows and the default view was used, optionally print a separate "Deferred" section so the user remembers they exist:

```
Deferred (not actionable):
  UF-NOTIF-005 | Member manages notification preferences | tags: settings
```

Skip the deferred section if it would be empty, or if a slug was given (the user is already filtering — don't add noise).

## Anti-patterns specific to pending

- Suggesting which flows the user should create tasks for. The list itself is the suggestion.
- Auto-generating tasks for the user. They use `task` for that.
- Falling back to the default view when an unrecognized slug is given. Print the slug table and stop — don't paper over a typo.
- Accepting comma-separated slugs or multiple args. One slug per invocation.
- Re-reading domain files for AC content. Read them only to extract the `Status:` line for each flow.
- Including the deferred breakdown when a slug was given. The user is already narrowing — keep the output tight.
