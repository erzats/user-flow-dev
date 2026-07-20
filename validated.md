# `/user-flow-dev validated <TASK-ID>`

Archive a task after the user has manually validated it. Run this only after `/user-flow-dev done` has staged the task as `needs manual validation` and the user has actually exercised the flow.

**Scope:** writes inside the resolved `FLOW_ROOT` only.

## When this runs

Almost always invoked by the user, not Claude. The user reads the manual-validation checklist that `done` wrote into the task file, exercises each item in the actual product, and then types `/user-flow-dev validated TASK-NNN` to close it out.

Claude may run this autonomously **only** when the user has explicitly said the validation passed (e.g. "I tested it, looks good — close it out"). Do not infer validation from anything else — not from the user saying "thanks", not from the user moving on to the next task, not from the absence of complaints. If you're not sure, ask: "Did you exercise the manual checklist? Should I run `validated` now?"

## Procedure

### Step 1 — Validate the input

`$ARGUMENTS` should be `validated TASK-NNN`. Anything else:

- No TASK ID → "Usage: `/user-flow-dev validated <TASK-ID>`. See `tasks/todo.md` for tasks awaiting validation." and stop.
- Multiple IDs → "One task per `validated` command." and stop.
- TASK ID not found in `todo.md` → check `tasks/archive/`. If already archived, respond "TASK-NNN is already archived." and stop. Otherwise: "No task with ID `<X>` in todo.md." and stop.
- Task status in `todo.md` is `pending` or `in-progress` (not yet staged by `done`) → "TASK-NNN is still in `<status>`. Run `/user-flow-dev done TASK-NNN` first to stage it for validation." and stop.

### Step 2 — Move the task file to archive

Move `tasks/<domain>/TASK-NNN.md` to `tasks/archive/TASK-NNN.md`. Inside the archived file, just under the `# TASK-NNN — ...` header, replace any existing `**Status:**` and `**Implementation completed:**` lines with:

```
**Archived:** <YYYY-MM-DD>
**Status:** done
**Validated by:** user
```

Leave the `## Manual validation` section in place. It's a useful record of what was checked.

### Step 3 — Update `todo.md`

Find the line `TASK-NNN | <summary> | <domain> | <FLOW-ID> | needs manual validation` and **remove it**. The archive directory is the record; `todo.md` stays focused on what's still active.

### Step 4 — Update the flow's status

Open the linked flow's domain file and locate the `## UF-<DOMAIN>-NNN — ...` section.

- Change the `Status:` line to `Status: completed`.
- Remove the `Active task: tasks/<domain>/TASK-NNN.md` line entirely.

Leave every other field untouched — AC, branches, dependencies, preconditions are not part of this command's scope.

### Step 5 — Confirm

Print:

```
TASK-NNN validated and archived.
- Moved tasks/<domain>/TASK-NNN.md → tasks/archive/TASK-NNN.md
- Removed from todo.md
- Flow UF-<DOMAIN>-NNN status → completed
- Removed Active task: line from flow detail
```

## Anti-patterns specific to validated

- Running `validated` on a task that's still `pending` or `in-progress`. The two-phase gate exists for a reason.
- Running `validated` autonomously based on a guess about whether the user tested. If they didn't say so, ask.
- Skipping the flow status update. A flow with all ACs holding *and* manual validation is `completed`; that's the whole point of this command.
- Modifying anything in the archived file beyond the three header lines noted in Step 2.
- Re-archiving a task already in `tasks/archive/`. Stop and report.
