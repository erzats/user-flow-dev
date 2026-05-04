# `/user-flow-dev done <TASK-ID>`

Mark a task done and archive its file. **Expected to be called automatically by Claude when finishing implementation work tied to a TASK-ID** — not just by the user.

**Scope:** writes inside `.claude/user-flows/` only.

## When Claude should call this autonomously

If the project's CLAUDE.md mentions user-flow-dev (it should, after `init`), then any time Claude completes implementation work that closes out a `TASK-NNN`, it should run this command without being asked. Specifically:

- Just merged a PR / committed code that completes a tracked task.
- Verified that the flow's acceptance criteria now hold (via `check` or by tracing specific code/tests in the same session).
- The task file in `tasks/<domain>/TASK-NNN.md` exists and matches the work done.

If the work is partial — only some branches done, AC not yet verified — do not call `done`. Update `todo.md`'s status column to `in-progress` instead (this is a manual edit, no separate command).

## Procedure

### Step 1 — Validate

`$ARGUMENTS` should be `done TASK-NNN`. Anything else:

- No TASK ID → "Usage: `/user-flow-dev done <TASK-ID>`. See `tasks/todo.md` for active tasks." and stop.
- Multiple IDs → "One task per `done` command." and stop.
- TASK ID not found in `todo.md` → check `tasks/archive/`. If already archived, respond "TASK-NNN is already archived." and stop. Otherwise: "No task with ID `<X>` in todo.md. Was it created with `/user-flow-dev task`?" and stop.

### Step 2 — Verify ACs were actually checked

Before archiving, the flow's acceptance criteria must be verified against code or tests — not assumed. The same rule from `check.md` applies: no vibes-based verification.

If you (Claude) just finished the implementation and traced specific code that enforces each AC in this same session, that counts. State which ACs you verified and where in the confirmation output (Step 5).

If you have not actually traced the ACs, **stop and run `/user-flow-dev check <domain>`** (or do the equivalent inline tracing) before archiving. Do not archive a task whose flow has unverified ACs.

If `check` reveals broken or unclear ACs, do not archive. Tell the user what's broken and let them decide whether to fix it (keep task `in-progress`), defer it, or override.

### Step 3 — Move the file

Move `.claude/user-flows/tasks/<domain>/TASK-NNN.md` to `.claude/user-flows/tasks/archive/TASK-NNN.md`.

Inside the archived file, just under the `# TASK-NNN — ...` header, ensure these two lines exist (replace the previous Status line if present):

```
**Archived:** <YYYY-MM-DD>
**Status:** done
```

### Step 4 — Update `todo.md`

Find the line `TASK-NNN | <summary> | <domain> | <FLOW-ID> | <status>` and **remove it**.

(The archive directory is the record of done work; `todo.md` stays focused on what's actually pending. If a project has a different convention, it can override — but the default is removal.)

### Step 5 — Confirm

Print:

```
TASK-NNN archived.
- Moved tasks/<domain>/TASK-NNN.md → tasks/archive/TASK-NNN.md
- Removed from todo.md
- Linked flow: UF-<DOMAIN>-NNN
- AC verification: <list each AC and the file:line that enforces it, OR "ran /user-flow-dev check <domain> — all ACs ✓">
```

The AC verification line is the load-bearing part of this output. If it says "verified" without specifics, the user can't trust it.

## Anti-patterns specific to done

- Deleting the task file instead of moving to archive. Archive is non-negotiable — completed work stays as a record.
- Marking a partial task done. If branches are still TODO, status is `in-progress`, not `done`.
- Calling `done` without verifying the flow's AC against actual code or tests. The whole point of the registry is the verification loop.
- Calling `done` on tasks that aren't yours to close (e.g. another agent or human's task you didn't actually finish).
- Stating "verified" in the confirmation without listing what was verified and where. If you can't list it, you didn't verify it.
