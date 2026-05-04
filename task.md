# `/user-flow-dev task <FLOW-ID>`

Generate a task file for **one** specific flow. **Never bulk-generates.**

**Scope:** all writes inside `.claude/user-flows/`.

## Hard rules

- One flow per invocation. If `$ARGUMENTS` after `task` contains multiple IDs or no ID, stop and explain.
- The flow must already exist. If the ID is not in `overview.md`, stop and suggest `/user-flow-dev add` instead.
- The task file references the flow ID. It does not restate flow content.

## Procedure

### Step 1 — Validate the input

`$ARGUMENTS` should be `task UF-<DOMAIN>-NNN`. Anything else:

- No ID → respond with "Usage: `/user-flow-dev task <FLOW-ID>`. Run `/user-flow-dev pending` to see flows without tasks." and stop.
- Multiple IDs → respond with "One flow per task command. Pick the one to start with." and stop.
- ID not found in `overview.md` → respond with "No flow with ID `<X>`. Did you mean `<closest match>`? Or run `/user-flow-dev add` to document it first." and stop.

### Step 2 — Load the flow detail

Read the matching domain file. Locate the `## UF-<DOMAIN>-NNN — ...` section. You need the full detail (branches + AC) to scope the task — even though the task file won't restate it.

### Step 3 — Pick a TASK ID

Read `tasks/todo.md` and `tasks/archive/`. Find the highest existing `TASK-NNN` across both. Increment by 1. Pad to 3 digits.

(TASK IDs are global across domains — there is one sequence, not one per domain. This makes them unambiguous in conversation.)

### Step 4 — Determine the domain folder

The task lives in `tasks/<domain>/`, where `<domain>` matches the flow's domain. If the folder doesn't exist, create it.

### Step 5 — Write the task file

Path: `.claude/user-flows/tasks/<domain>/TASK-NNN.md`

Shape:

```markdown
# TASK-NNN — <one-line summary>

**Flow:** UF-<DOMAIN>-NNN ([detail](../../domains/<domain>.md#uf-<domain>-nnn--<title-slug>))
**Status:** pending
**Created:** <YYYY-MM-DD>

## Scope

<2–4 bullets capturing what's actually being built. Reference the flow's branches and AC — do NOT restate them. Example: "Implements main path + the coupon branch" is enough.>

## Out of scope

<Optional. Note any branches deferred to a later task.>

## Verification

When done, the flow's acceptance criteria must hold. Run `/user-flow-dev check <domain>` (or manually verify each AC against code/tests) before marking this task done.

## Notes

<Optional space for implementation hints, dependencies on other tasks, or open questions. Keep this lean.>
```

### Step 6 — Append to `todo.md`

Append one line at the bottom (don't reorder existing lines):

```
TASK-NNN | <summary> | <domain> | UF-<DOMAIN>-NNN | pending
```

The summary in `todo.md` matches the `# TASK-NNN — <summary>` header in the task file. They must agree.

### Step 7 — Confirm

Print:

```
Created TASK-NNN at .claude/user-flows/tasks/<domain>/TASK-NNN.md
Linked to UF-<DOMAIN>-NNN.
Marked pending in todo.md.
```

## Anti-patterns specific to task

- Restating the flow's branches or acceptance criteria inside the task file. Reference the ID; that's the point.
- Generating tasks for multiple flows in one command. One per invocation.
- Auto-marking the task `in-progress` on creation. It starts `pending` until the user (or Claude) actually starts work and updates it.
- Skipping the existence check. A typo'd ID should error, not silently create a task pointing to nothing.
- Using a per-domain TASK number (e.g. TASK-AUTH-001). TASK numbers are global.
