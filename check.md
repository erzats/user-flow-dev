# `/user-flow-dev check [domain]`

Verify documented acceptance criteria still hold against the current code, write each examined flow's `Status:`, and reconcile its task lifecycle (create / update / archive).

`check` is the single command responsible for keeping flow status and task state in sync with the codebase. After a `check` run, every examined flow has the right status and the right task (or no task, when `completed`).

## STOP — no vibes-based verification

The whole point of `check` is to be the verification loop the rest of the system relies on. If you skim a file, see something that "looks roughly right", and mark an AC as `✓ holds`, you have actively destroyed the value of this skill. The user's project will silently rot while the report says everything is fine.

**An acceptance criterion is verified only if you traced specific code (or a specific test) that demonstrably enforces it.** If you cannot trace it in 60 seconds of focused search, the verdict is `⚠ unclear`, never `✓ holds`. There is no in-between.

The cost of a false positive (claiming a broken flow works) is much higher than a false negative (flagging a working flow as unclear). Calibrate accordingly.

## Scope

- `check` (no arg) — checks every active flow across all domains. Slow. Use before a release. Warn the user about runtime up front.
- `check <domain>` — checks only flows in one domain. Use before merging a feature branch that touched that area.

Skip flows whose status is `deferred` or `superseded by ...`. Those encode product/refactoring intent that `check` has no signal for and must not overwrite. Examine all other flows: `init`, `not started`, `incomplete`, `issues`, `needs manual validation`, `completed`. For these statuses, `check` will rewrite the value based on the new verdict.

## Procedure

### Step 1 — Load the target flows

If a domain was specified:
- Read `.claude/user-flows/domains/<domain>.md` only.
- If the domain file doesn't exist, list available domains from `overview.md` and stop.

If no domain was specified:
- Read `overview.md` to enumerate domains.
- Then read each `.claude/user-flows/domains/*.md` in turn.
- Warn the user: "Checking N flows across M domains; this will take a while."

For each flow, extract:
- Flow ID and title
- Surface (web/mobile/both/admin-only) — narrows where to search
- Acceptance criteria (the bullet list under `### Acceptance criteria`)
- Tags — additional search hints
- The `Active task:` line if present (points to an existing open task for this flow)

### Step 2 — Locate the implementation

For each flow, find the area that implements it. Use:

- The flow's surface to pick the right app folder (`apps/web/`, `apps/mobile/`, both)
- The domain name as a search hint
- Tags as additional search hints
- The flow title's nouns and verbs

Use Grep and Read. For a thorough sweep across a large area, dispatch the Explore agent. Do NOT guess — if you cannot locate the implementation with a few targeted searches, the verdict for that flow's ACs is `– not implemented` or `⚠ unclear` (whichever fits). Inability to find code is itself useful signal.

### Step 3 — Evaluate each acceptance criterion

For each AC bullet, decide one of:

| Verdict | Meaning | When to use |
|---|---|---|
| `✓ holds` | Code or tests demonstrably enforce the criterion. **Required:** cite a file path and a function or line range. | Only when you traced specific code that enforces the AC. No exceptions. |
| `⚠ unclear` | Implementation exists but you can't tell from static reading whether the AC holds. | When tracing was inconclusive — async behavior, runtime config, untested branches. Default to this when in doubt. |
| `✗ broken` | Code or tests demonstrably violate the criterion. | When you can point to specific code that contradicts the AC. Cite where and explain. |
| `– not implemented` | The flow's implementation isn't present at all. | When no code corresponds to the flow. Distinct from `unclear`. |

A `✓ holds` without a file:line citation is invalid — downgrade it to `⚠ unclear`.

### Step 4 — Compute per-flow status

For each examined flow, compute the new status from its AC verdicts (highest priority wins):

| Per-flow condition | New status |
|---|---|
| Any AC `✗ broken` | `issues` |
| Any AC `⚠ unclear` (and none broken) | `needs manual validation` |
| Mix of `✓ holds` and `– not implemented` (no broken, no unclear) | `incomplete` |
| All ACs `– not implemented` | `not started` |
| All ACs `✓ holds` | `completed` |

### Step 5 — Read the task baseline for each flow

Before writing anything, look up each examined flow's existing task state. This is what lets `check` distinguish "still broken since last week" from "newly broken" and avoid no-op writes.

For each flow:

1. **Open task?** Search `tasks/<domain>/TASK-*.md` for a file whose `**Flow:** UF-<DOMAIN>-NNN` matches. Read it and capture:
   - Its `**Source:**` (`manual` or `check`)
   - Its `## Findings` section if present (the existing verdicts and verdict line)
2. **Archived task?** Search `tasks/archive/TASK-*.md` for files whose `**Flow:**` matches. Capture the most recent (highest TASK-NNN) and its archive date. Used as a regression breadcrumb if a new task is created.
3. **No task?** Note that — the flow has no history.

This is read-only. No writes happen in this step.

### Step 6 — Print the report

Format:

```
## Check report — <scope>

### auth/UF-AUTH-001 — Sign in with SSO
- ✓ Sign-in completes in under 5 seconds — apps/web/src/app/auth/route.ts:42-78
- ✓ Cancelling OAuth never produces an error toast — apps/mobile/screens/SignIn.tsx:114
- ⚠ Session persists across app restarts — needs runtime test; AsyncStorage logic at apps/mobile/contexts/AuthContext.tsx:60 looks correct but expiry handling is unclear

### payments/UF-PAYMENTS-002 — Subscribe via Stripe Checkout
- ✓ App state never says "subscribed" before Stripe confirms — apps/web/src/app/api/stripe-webhook/route.ts:30
- ✗ Successful payment converges to "subscribed" within 5 minutes — no reconciliation job exists. Webhook is the only path; if dropped, state never converges. (apps/web/src/app/api/stripe-webhook/route.ts handles only the webhook)

[continued for each flow]

---

## Summary

Flows checked: 18
- 14 fully passing (every AC ✓)
- 2 with unclear ACs (manual verification needed)
- 1 with broken ACs (UF-PAYMENTS-002)
- 1 with no implementation (UF-NOTIF-005)

## Task changes
- TASK-051 created (UF-PAYMENTS-002 — issues)
- TASK-042 updated (UF-NOTIF-005 — findings refreshed)
- TASK-038 archived (UF-AUTH-001 — flow now completed)
- TASK-039 unchanged (UF-PAYMENTS-001 — findings identical to last run)
```

The "Task changes" block is non-negotiable. The user must be able to see exactly what was created, updated, archived, or skipped without reading the diff.

### Step 7 — Write status updates to domain files

For each examined flow, edit the `Status:` line in the domain file to the new value computed in Step 4. Edit only that line. Do not touch AC text, branches, dependencies, or any other field.

If the new status equals the existing status, you may still rewrite (idempotent) or skip — both are fine.

If a flow had `Status: deferred` or `Status: superseded by ...`, you should not be in this step at all (those flows were skipped at Step 1).

### Step 8 — Reconcile the task for each flow

For each examined flow, decide what to do with its task based on the new status and the baseline you read in Step 5:

| New flow status | Baseline (from Step 5) | Action |
|---|---|---|
| `completed` | open task exists | **Archive it.** Move `tasks/<domain>/TASK-NNN.md` → `tasks/archive/TASK-NNN.md`, stamp `**Archived:** <date>` and `**Status:** done`, remove the line from `todo.md`, remove the `Active task:` line from the flow detail. |
| `completed` | no open task | Nothing to do. |
| `not started` | open task exists | Leave it alone (the user opened it intentionally; just-started work is a normal state). |
| `not started` | no open task | Nothing to do. (`not started` flows are pre-implementation; no findings to record yet.) |
| `incomplete` / `issues` / `needs manual validation` | open task exists, findings unchanged | Skip the write. Idempotent. |
| `incomplete` / `issues` / `needs manual validation` | open task exists, findings changed | Rewrite the `## Findings` section in place. Leave every other section (including `## Notes`) untouched. |
| `incomplete` / `issues` / `needs manual validation` | no open task, archived task exists for this flow | Create a new task. In the `## Findings` section, include a `**Previously:** TASK-XXX (archived YYYY-MM-DD)` line as a regression breadcrumb. |
| `incomplete` / `issues` / `needs manual validation` | no open task, no archived task | Create a fresh task. |

For "skip the write", compare findings text-equivalent: same verdict line, same per-AC bullets (verdict + reason + citation). The `**Last checked:**` date is not part of the comparison.

#### When creating a new task

Pick a TASK ID by reading `tasks/todo.md` and `tasks/archive/`, finding the highest existing `TASK-NNN`, and incrementing by 1. Pad to 3 digits.

Write `tasks/<domain>/TASK-NNN.md` with this shape:

```markdown
# TASK-NNN — UF-<DOMAIN>-NNN findings: <flow title>

**Flow:** UF-<DOMAIN>-NNN ([detail](../../domains/<domain>.md#uf-<domain>-nnn--<title-slug>))
**Status:** pending
**Created:** <YYYY-MM-DD>
**Source:** check

## Findings

**Last checked:** <YYYY-MM-DD>
**Verdict:** <issues | needs manual validation | incomplete>

- <verdict glyph> AC text — file:line citation or reason
- <verdict glyph> AC text — file:line citation or reason
...

**Previously:** TASK-XXX (archived YYYY-MM-DD)   ← include only if there's a prior archived task for this flow

## Notes

(user-editable; `/user-flow-dev check` never touches this section)
```

Then append one line to `todo.md`:

```
TASK-NNN | findings: <flow title> | <domain> | UF-<DOMAIN>-NNN | pending
```

And add an `Active task:` line to the flow detail in the domain file, immediately under the `Status:` line:

```
Status: issues
Active task: tasks/<domain>/TASK-NNN.md
```

#### When updating an existing task

Locate its `## Findings` section (anywhere in the file) and rewrite that section only, in the format above. Leave every other section — `## Scope`, `## Out of scope`, `## Verification`, `## Notes`, anything custom — untouched.

If the existing task does not yet have a `## Findings` section (e.g. a manual task created before its first check), insert one. Place it after `## Verification` if present, otherwise immediately before `## Notes`. If neither anchor exists, append it to the end of the file.

The `## Findings` section is the only thing `check` writes inside a task body, ever.

#### When archiving (flow now completed)

Move the task file to `tasks/archive/TASK-NNN.md`. Inside the archived file, just under the `# TASK-NNN — ...` header, ensure these two lines exist (replacing any prior `**Status:**` line):

```
**Archived:** <YYYY-MM-DD>
**Status:** done
```

Remove the line for this task from `todo.md`. Remove the `Active task:` line from the flow detail in the domain file.

This is the same archival shape as `/user-flow-dev done`, just initiated by `check` because the verification loop confirmed the flow is now completed.

### Step 9 — Do not modify anything else

`check` writes:
- `Status:` and `Active task:` lines in flow detail sections
- One task file per examined non-completed flow (created or updated)
- `todo.md` (append on create, remove on archive)
- The archive folder (on archive)

It does not:
- Edit AC text, branches, preconditions, or any other flow field. Those change via deliberate edits, not as a side effect of check.
- Touch `## Notes` or any non-`## Findings` section inside a task file.
- Stamp a "last checked" timestamp anywhere outside the task `## Findings` section.
- Flip `deferred` or `superseded` flows to anything else — those are off-limits.

## Anti-patterns specific to check

- Reasoning about ACs against the AC text alone, without searching the codebase. The check is against code, not against itself.
- Citing a file without a line range or function. A bare filename is not a citation.
- Modifying any flow field other than `Status:` and `Active task:`. ACs and other content change deliberately, not as a side effect.
- Touching `deferred` or `superseded` flows.
- Touching anything inside a task file other than the `## Findings` section. Manual tasks belong to the user; `## Notes` belongs to the user.
- Skipping the baseline read (Step 5). Without it, `check` rewrites identical findings on every run and can't tell a regression apart from a recurring problem.
- Reopening an archived task to record a new regression. Archived tasks are historical; create a new task and link the archived one as `**Previously:**`.
- Treating `check` (no arg) as quick. It is not. Warn up front.
