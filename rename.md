# `/user-flow-dev rename <domain> <new-name>`

Rename a single domain. Preview-then-confirm; never silent. The simplest of the three refactor commands — flow numbers are preserved, only the prefix changes.

**Scope:** all writes are confined to `.claude/user-flows/`. The rename touches the domain file (renamed in place via a new file + tombstone-converted old file), `overview.md`, the `tasks/<domain>/` folder (renamed), `tasks/todo.md`, and any flow/task that referenced a UF-<OLDNAME>-NNN ID.

## When to use it

A domain's name no longer fits — the original inference was wrong, or the project's terminology has evolved. Rename when the **set of flows isn't changing**, only the label.

If you're combining with another domain, use `merge`. If you're separating concerns inside the domain, use `split`. The new name must not collide with an existing domain — if `domains/<new-name>.md` exists, refuse and tell the user to use `merge` instead.

## Phase 1 — Propose

### Step 1: Validate

Read `.claude/user-flows/domains/<domain>.md`. Confirm:

- The source domain exists.
- The new name is lowercase and hyphenated. (`identity`, not `Identity` or `identity_management`.)
- `.claude/user-flows/domains/<new-name>.md` does not exist.
- The new name aligns with project terminology — the README, CLAUDE.md, or top-level folders should suggest it. If the new name has no support in the project's vocabulary, ask the user to confirm before continuing.

If any structural check fails (collision, illegal name, source not found), stop and tell the user.

### Step 2: Show the rename plan

```
Rename: auth → identity

Files:
  domains/auth.md → domains/identity.md (new file written; old file kept as tombstone)
  tasks/auth/ → tasks/identity/

Flow IDs (numbers preserved, prefix changes):
  UF-AUTH-001 → UF-IDENTITY-001  Sign up with email + password
  UF-AUTH-002 → UF-IDENTITY-002  Sign in with SSO
  UF-AUTH-003 → UF-IDENTITY-003  Password reset
  ...

Tombstones in domains/auth.md (kept as a tombstones-only file): N
Tasks affected: M
```

End with: "Confirm or cancel."

**Stop and wait.** Do not write anything in Phase 1.

## Phase 2 — Execute

### Step 3: Write the new domain file

Create `.claude/user-flows/domains/<new-name>.md` containing the source domain's full content with all UF-IDs renumbered (number preserved, prefix changed). Process in order:

- Copy the source's 2-sentence summary verbatim. The flows haven't changed, only the name.
- Rewrite the domain's flow index — replace every `UF-<OLDNAME>-NNN` with `UF-<NEWNAME>-NNN`.
- Rewrite each flow detail heading to the new ID.
- Add a line directly under `Status:` reading `Renamed from: UF-<OLDNAME>-NNN` for every flow.
- Update each `Active task:` path to point to `tasks/<new-name>/TASK-NNN.md`.

### Step 4: Convert the old domain file to tombstones-only

Replace `.claude/user-flows/domains/<old-name>.md` with:

```markdown
# <old-name> (renamed)

> Domain renamed to `<new-name>`. See [domains/<new-name>.md](<new-name>.md) for current flows. Tombstones below preserve old IDs for external references.

## Tombstones

UF-<OLDNAME>-001 | <Title> (renamed) → UF-<NEWNAME>-001 | <old-name> | tags: ...
UF-<OLDNAME>-002 | <Title> (renamed) → UF-<NEWNAME>-002 | <old-name> | tags: ...
...

---

## UF-<OLDNAME>-001 — <Title> (renamed)

Status: moved to UF-<NEWNAME>-001

## UF-<OLDNAME>-002 — <Title> (renamed)

Status: moved to UF-<NEWNAME>-002

...
```

The old file is **not deleted**. It is the redirect surface for external references.

### Step 5: Move the task folder

- Rename `tasks/<old-name>/` to `tasks/<new-name>/` (folder rename; contents come along).
- For each task file inside the renamed folder, update its `**Flow:** UF-<OLDNAME>-NNN` line to the new ID.
- Update `tasks/todo.md`: change the domain column and FLOW-ID column for every affected line.

### Step 6: Update overview.md

- Domains list: replace the entry for the old name with the new name. Annotate the new entry `(renamed from <old-name>)` so the rename is discoverable from `overview.md`.
- Add a small entry below for the old name marked `→ renamed to <new-name>` so a reader scanning for the old name finds the redirect.
- Flows section: the old name's subsection now contains tombstone lines; the new name's subsection contains the renumbered flow lines.

### Step 7: Auto-rewrite references

Scan `.claude/user-flows/**/*.md` (excluding the old domain file's tombstones and `Renamed from:` lines) for occurrences of `UF-<OLDNAME>-` and replace with `UF-<NEWNAME>-`. Surface a one-line summary of every file rewritten, same format as `split.md` Step 11.

Do not touch files outside `.claude/user-flows/`.

### Step 8: Confirm

Print a one-paragraph summary including:

- Old name → new name
- Total flows renumbered, tombstones created, tasks moved, references rewritten
- The exact list of file paths created, modified, or moved
- Suggested next step: usually nothing — rename is a pure relabel, so flow status is unaffected. If the rename was prompted by a real change in scope, run `/user-flow-dev check <new-name>` to make sure ACs still match the code.

Listing every file path is non-negotiable.

## Anti-patterns specific to rename

- Renaming into an existing domain name. That's a merge — refuse and route the user to `/user-flow-dev merge`.
- Skipping the tombstone file. External references need somewhere to land.
- Forgetting to rename the tasks folder.
- Renaming because the user wants a prettier name without checking the new name actually appears in project terminology. The skill should ask before renaming to a name that isn't supported by the README or CLAUDE.md.
- Writing files in Phase 1.
