# `/user-flow-dev merge <domain1> <domain2> [into <target>]`

Refactor two domains into one. Conversational; never one-shot. Phase 1 proposes the merge target and ID renumbering; Phase 2 executes after the user confirms.

**Scope:** all writes are confined to `.claude/user-flows/`. The merge touches both domain files, `overview.md`, `tasks/todo.md`, task file paths, and any flow/task that referenced a renumbered UF-ID.

## When to use it

Two domains have turned out to be the same concern — actors, lifecycle, and tags overlap heavily, and readers keep loading both files together. Or one domain is genuinely thin (<3 flows with no distinct-actor / lifecycle / risk exception) and its closest neighbor is the natural home.

Don't merge unrelated domains just because both are small. The skill's whole point is domain-organized scannability; merging unrelated concerns hurts that. If two thin domains aren't related, leave them thin.

## Phase 1 — Propose (no file writes yet)

### Step 1: Read both sources

Read both `.claude/user-flows/domains/<domain1>.md` and `<domain2>.md` in full, plus `overview.md`. If `$ARGUMENTS` includes an `into <target>` clause, use the named target as the proposal seed. Otherwise pick a default (Step 2).

### Step 2: Pick the target domain name

Default rule: if one domain has clearly more flows or more central project terminology, the smaller domain merges into it (the larger one's name wins). If they're similar size or both names are partial, propose a third name that captures the union (e.g. `auth` + `sso` → `identity`).

Confirm the target name with the user before continuing. The user may override.

### Step 3: Propose ID renumbering

Flows from a **source** domain that's being absorbed get renumbered with the target's prefix, picking up after the target's max existing number. Flows that already live in the target keep their IDs.

```
Merge: sso → auth (target: auth)

Renumber (sso → auth):
  UF-SSO-001 → UF-AUTH-008  Sign in with SSO
  UF-SSO-002 → UF-AUTH-009  First-time SSO provisioning
  UF-SSO-003 → UF-AUTH-010  SSO email conflict resolution

Keep (auth):
  UF-AUTH-001 ... UF-AUTH-007  no change
```

If both sources merge into a third name (e.g. `auth` + `sso` → `identity`), all flows from both sources renumber.

### Step 4: Present and wait

Output:

- Source domains → target name
- Full ID renumbering table (one section per source)
- Tombstone count (= number of flows that changed ID)
- Task files that will move
- Disposition of each absorbed source domain file: converted to a tombstones-only file (not deleted)
- A note that internal references will be auto-rewritten

End with: "Confirm, edit, or cancel before I write files."

**Stop and wait.** Do not write anything in Phase 1.

## Phase 2 — Execute

### Step 5: Append flows to the target

For each renumbered flow:

- Move its detail section into the target domain file (creating it if the target name is new).
- Update the heading to the new ID: `## UF-<TARGET>-NNN — <Title>`.
- Add `Renamed from: UF-<OLDDOMAIN>-NNN` directly under `Status:`.
- Append the flow's index line to the target's index with the new ID.
- Refresh the target's 2-sentence summary if the merge changed its scope.

For flows that already live in the target with no ID change, do nothing.

### Step 6: Convert each absorbed domain file to tombstones-only

For each domain that's being absorbed (i.e. its name is going away):

- Replace the 2-sentence summary at the top of the file with:

  ```markdown
  > Domain merged into `<target>`. See [domains/<target>.md](<target>.md) for current flows. Tombstones below preserve old IDs for external references.
  ```

- Replace each flow detail with a tombstone:

  ```markdown
  ## UF-<OLDDOMAIN>-NNN — <Title> (moved)

  Status: moved to UF-<TARGET>-NNN
  ```

- Update the absorbed domain's index: every entry becomes `<old-id> | <Title> (moved) → <new-id> | <old-domain> | tags: ...` so a reader scanning the index sees redirects immediately.

The absorbed domain file is **not deleted**. It remains as a redirect surface.

### Step 7: Move task files

For each task whose flow moved domains:

- Move `tasks/<old-domain>/TASK-NNN.md` to `tasks/<target>/TASK-NNN.md`. Create the destination folder if needed.
- Update the task's `**Flow:** UF-<OLDDOMAIN>-NNN` line to the new ID.
- Update `tasks/todo.md`: change the domain column and FLOW-ID column for that line.
- Update the `Active task:` line in the target's flow detail to the new path.

### Step 8: Update overview.md

- Domains list: keep the absorbed domain's entry but annotate it `→ merged into <target>`. Add or refresh the target's entry. Do not remove the absorbed entry — it makes the redirect discoverable from `overview.md` alone.
- Flows section: the absorbed domain's subsection now contains tombstone lines (one per flow with `(moved) → <new-id>`); the target's subsection grows with the renumbered flows under their new IDs.

### Step 9: Auto-rewrite references

Scan `.claude/user-flows/**/*.md` (excluding tombstone blocks and `Renamed from:` lines) for renumbered UF-IDs and update them. Surface a one-line summary of every file rewritten, same format as `split.md` Step 11.

Do not touch files outside `.claude/user-flows/`.

### Step 10: Confirm

Print a one-paragraph summary including:

- Source domains → target
- Total flows merged, tombstones created, tasks moved, references rewritten
- The exact list of file paths created, modified, or moved
- Suggested next step: `/user-flow-dev pending` to confirm flow status didn't drift, or `/user-flow-dev check <target>` if substantial code edits accompanied the merge

Listing every file path is non-negotiable.

## Anti-patterns specific to merge

- Merging unrelated domains because one is thin. Leave thin or fold into a related neighbor.
- Merging into a name that obscures both sources. The target name should make sense to a reader who only sees the target file.
- Deleting the absorbed domain file outright. Convert it to tombstones-only — external references rely on it.
- Auto-rewriting tombstones or `Renamed from:` lines.
- Writing files in Phase 1.
