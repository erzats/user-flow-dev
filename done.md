# `/user-flow-dev done <TASK-ID>`

Mark a task's implementation complete and stage it for manual validation. **Expected to be called automatically by Claude when finishing implementation work tied to a TASK-ID** — not just by the user.

`done` does **not** archive the task. It flips the task and the linked flow to `needs manual validation` and leaves both open until the user runs `/user-flow-dev validated <TASK-ID>` after eyeballing the change. Archive happens there, never here. The default is two-phase because code-tracing can't prove a screen looks right or an interaction feels right; even pure helper code has downstream effects.

The escape hatch (`--skip-validation`) exists for the rare task where Claude can articulate, in one sentence, why no user-observable behavior could possibly change. Use it almost never.

**Scope:** writes inside `.claude/user-flows/` only.

## When Claude should call this autonomously

After completing implementation that closes a `TASK-NNN` — committed code matches the task and the flow's ACs are verified against code (via `check` or inline tracing in the same session). If branches are still TODO or ACs aren't verified, leave the task `in-progress` (edit `todo.md`); don't call `done`.

## Procedure

### Step 1 — Validate the input

`$ARGUMENTS` should be `done TASK-NNN`, optionally followed by `--skip-validation "<reason>"`.

- No TASK ID → "Usage: `/user-flow-dev done <TASK-ID>`. See `tasks/todo.md` for active tasks." and stop.
- Multiple IDs → "One task per `done` command." and stop.
- TASK ID not found in `todo.md` → check `tasks/archive/`. If already archived, respond "TASK-NNN is already archived." and stop. Otherwise: "No task with ID `<X>` in todo.md. Was it created with `/user-flow-dev task`?" and stop.
- `--skip-validation` provided without a reason string → "`--skip-validation` requires a one-sentence justification of why no user-observable behavior could change. Stop and reconsider — almost every task needs manual validation." and stop.

### Step 2 — Verify ACs were actually code-traced

Before changing anything, the flow's acceptance criteria must be verified against code or tests — not assumed. The same rule from `check.md` applies: no vibes-based verification.

If you (Claude) just finished the implementation and traced specific code that enforces each AC in this same session, that counts. State which ACs you traced and where in the confirmation output (Step 6).

If you have not actually traced the ACs, **stop and run `/user-flow-dev check <domain>`** (or do the equivalent inline tracing) before continuing. Do not stage a task whose ACs are unverified.

If `check` reveals broken or unclear ACs, do not stage. Tell the user what's broken and let them decide whether to fix it (keep task `in-progress`), defer it, or override.

### Step 3 — Pick the destination state

Decide which path applies:

- **Default path: `needs manual validation`.** Almost always. The task represents real product behavior the user will see or interact with.
- **Skip-validation path: archive immediately.** Only if `--skip-validation "<reason>"` was provided AND the reason genuinely articulates why no user-observable behavior could change. Examples that qualify: a lint-rule added to CI; a deleted dead file with no imports; a renamed internal type with zero call-site impact. Examples that do **not** qualify: a refactored helper "with full test coverage"; a server action with new validation; anything in `apps/*/components/`, `apps/*/app/`, or `apps/*/screens/`.

If the path is unclear, default to `needs manual validation`. The cost of asking the user to spend 30 seconds clicking through is much lower than the cost of archiving a regression.

### Step 4a — Default path: stage for validation

Do these in order:

1. **Update the task file** at `tasks/<domain>/TASK-NNN.md`. Replace the existing `**Status:**` line with:

   ```
   **Status:** needs manual validation
   **Implementation completed:** <YYYY-MM-DD>
   ```

   Append (or create) a `## Manual validation` section just before `## Notes` (or at the end if no `## Notes`):

   ```markdown
   ## Manual validation

   Implementation is complete and ACs were code-traced. The user should now exercise the flow end-to-end, then run `/user-flow-dev validated TASK-NNN` (or report what they found if it's broken).

   Suggested checks:
   - <one bullet per AC that benefits from human verification — derived from the flow detail>
   - <add bullets for surfaces this change touched: which screen, which form, which mobile flow>
   ```

   Be specific in the bullets — generic "test the feature" is useless. If the flow has 4 ACs and you traced 2 cleanly in code, list the other 2 here.

2. **Update `todo.md`.** Change the status column on the `TASK-NNN | ... | <status>` line from `pending` / `in-progress` to `needs manual validation`. Do not remove the line. (Removal happens in `validated`.)

3. **Update the flow's domain file.** Locate `## UF-<DOMAIN>-NNN — ...`. Change its `Status:` line to `Status: needs manual validation`. Leave the `Active task:` line in place.

### Step 4b — Skip-validation path: archive immediately

This path only runs if `--skip-validation "<reason>"` was provided.

Move `tasks/<domain>/TASK-NNN.md` to `tasks/archive/TASK-NNN.md`. Inside the archived file, just under the `# TASK-NNN — ...` header, replace any existing `**Status:**` line with:

```
**Archived:** <YYYY-MM-DD>
**Status:** done
**Validation skipped:** <reason>
```

Remove the line for this task from `todo.md`. Remove the `Active task:` line from the flow detail in the domain file.

### Step 5 — Confirm

For the **default path**, print:

```
TASK-NNN staged for manual validation.
- Task status: needs manual validation
- Flow UF-<DOMAIN>-NNN status: needs manual validation
- AC code-trace: <list each AC and the file:line that enforces it, OR "ran /user-flow-dev check <domain> — all ACs ✓">

Manual validation checklist (in tasks/<domain>/TASK-NNN.md):
- <bullet>
- <bullet>

Run `/user-flow-dev validated TASK-NNN` after you've exercised the flow.
```

For the **skip-validation path**, print:

```
TASK-NNN archived (validation skipped).
- Moved tasks/<domain>/TASK-NNN.md → tasks/archive/TASK-NNN.md
- Removed from todo.md
- Removed Active task: line from UF-<DOMAIN>-NNN flow detail
- Skip reason: <reason>
- AC code-trace: <list each AC and the file:line that enforces it>
```

The AC code-trace line is the load-bearing part of the output. If it says "verified" without specifics, the user can't trust it.

## Anti-patterns specific to done

- Archiving on the default path. `done` no longer archives by default. Archive happens via `validated` (or the explicit skip-validation flag).
- Reaching for `--skip-validation` because manual validation feels like friction. The friction is the point — it's the gate that catches downstream effects.
- Marking a partial task `needs manual validation`. If branches are still TODO, status is `in-progress`, not staged.
- Calling `done` without code-tracing the ACs. The code-trace gate isn't optional — it's how `done` earns the right to ask the user to validate.
- Calling `done` on tasks that aren't yours to close.
- Stating "verified" without listing what was verified and where.
- Writing a generic manual-validation checklist ("test the feature"). Each bullet should name a screen, a form, a button, an interaction — something the user can actually do.
