# `/user-flow-dev split <domain> [hint]`

Refactor one domain into two or more by reassigning its flows. Conversational; never one-shot. Phase 1 proposes the split and waits; Phase 2 executes after the user confirms.

**Scope:** all writes are confined to `.claude/user-flows/`. The split touches the source domain file, creates new domain files, updates `overview.md`, moves task files, updates `tasks/todo.md`, and auto-rewrites references to renumbered UF-IDs across the registry.

## When to use it

A domain has grown past comfortable index size (typically >12 flows), or the flows inside it have separated into clearly distinct concerns — different actors, different lifecycle, or different risk profile. Don't split preemptively. A domain with 8 cohesive flows is fine.

If the source domain has fewer than 6 flows, splitting it is almost always wrong. Surface that to the user and ask before continuing — usually the right move is a `merge` into a neighbor instead.

## Phase 1 — Propose (no file writes yet)

### Step 1: Read the source

Read `.claude/user-flows/domains/<domain>.md` in full, plus `overview.md` for cross-references. If `$ARGUMENTS` includes a hint after the domain name (e.g. `split auth into sso and password`), use the named target domains as the proposal seed.

### Step 2: Group the flows

Cluster the source domain's flows by:

1. **Actors** — who experiences the flow. Admin vs end user vs system actor.
2. **Lifecycle** — onboarding vs steady-state vs offboarding.
3. **Tag cohesion** — flows that share 2+ tags likely belong together.
4. **Risk profile** — fraud-adjacent or security-critical flows belong together even if small.

Aim for 2–3 resulting domains. More than 3 usually means the original domain was a misnomer rather than a fat domain — flag that to the user.

### Step 3: Validate sizes

Each resulting domain must have **≥3 flows**. If a candidate domain has 1–2 flows, ask the user whether the **distinct actors / lifecycle / risk profile** exception applies. If not, fold those flows back into one of the larger candidates. The skill must not silently create thin domains via split.

### Step 4: Propose ID renumbering

Each flow that moves gets a new ID prefix matching its new domain. Numbers in a new domain start at 001 and append. If one of the resulting domains keeps the source's name, flows that stay there keep their original IDs (no renumbering for stayers).

```
Split: auth → auth (kept), sso (new)

Stay in auth/ (no renumber):
  UF-AUTH-001  Sign up with email + password
  UF-AUTH-003  Password reset
  UF-AUTH-006  Sign out

Move to sso/ (renumber):
  UF-AUTH-002 → UF-SSO-001  Sign in with SSO
  UF-AUTH-005 → UF-SSO-002  First-time SSO provisioning
  UF-AUTH-009 → UF-SSO-003  SSO email conflict resolution
```

If both resulting domains take new names (the source name disappears), every flow renumbers and the source domain file becomes a tombstones-only file (see Step 8 / Phase 2 Step 6 in `merge.md` for the same pattern).

### Step 5: Present and wait

Output a single block with:

- Source → target domains
- Per-target flow list with old → new IDs (or "no change" for stayers)
- Tombstone count (= number of flows that moved out of the source)
- Task files that will move with their flows
- A note that internal references will be auto-rewritten

End with: "Confirm, edit, or cancel before I write files. You can rename a target domain, reassign a flow, or fold a flow back."

**Stop and wait.** Do not write anything in Phase 1.

## Phase 2 — Execute (after user confirmation)

### Step 6: Create new domain files

For each target domain that doesn't already exist:

- Create `.claude/user-flows/domains/<new-domain>.md`.
- Write a fresh 2-sentence summary describing the new domain's role. Do not copy the source's summary.
- Add an empty index header, ready for moved flows.

### Step 7: Move flow detail sections

For each flow that's moving:

- Cut the flow detail block from the source domain file.
- Paste it into the new domain file under the renumbered heading: `## UF-<NEWDOMAIN>-NNN — <Title>`.
- Add a line directly under `Status:` reading `Renamed from: UF-<OLDDOMAIN>-NNN`.
- Append the flow's index line to the new domain's index using the new ID.
- Update the flow's `Active task:` line (if present) to the new task path (Step 9).

### Step 8: Add tombstones in the source

For each flow that left, replace its detail section in the source domain file with a tombstone:

```markdown
## UF-<OLDDOMAIN>-NNN — <Title> (moved)

Status: moved to UF-<NEWDOMAIN>-NNN
```

Update the source domain's index: change the index line for each moved flow to append `(moved) → UF-<NEWDOMAIN>-NNN`.

If the source domain now has zero live flows, convert it to a tombstones-only file (top-level note pointing to the new domains, tombstone entries below). Do not delete it — external references rely on it.

### Step 9: Move task files

For each task whose flow moved:

- Move `tasks/<old-domain>/TASK-NNN.md` to `tasks/<new-domain>/TASK-NNN.md`. Create the destination folder if needed.
- Update the task's `**Flow:** UF-<OLDDOMAIN>-NNN` line to the new ID.
- Update `tasks/todo.md`: change the domain column and FLOW-ID column for that line.
- Update the `Active task:` line in the new flow detail to point to the new path.

If a task references a flow that didn't move, leave it alone.

### Step 10: Update overview.md

- Update the Domains list: keep the source domain entry if it still has live flows (refresh its 2-sentence summary if scope changed); add an entry for each new domain.
- Update the Flows section: each flow's index line now lives under its new domain's section. Tombstone lines stay under the source domain's section so a reader scanning `overview.md` for an old ID still finds the redirect.

### Step 11: Auto-rewrite references

Scan `.claude/user-flows/**/*.md` (excluding the tombstone blocks just written in Step 8 and the `Renamed from:` lines from Step 7) for occurrences of any renumbered UF-ID. Replace with the new ID.

Surface a one-line summary of every file rewritten:

```
Rewrote UF-AUTH-002 → UF-SSO-001 in:
  domains/payments.md (1 reference)
  tasks/payments/TASK-014.md (2 references)
```

Do not rewrite tombstone blocks or `Renamed from:` lines — those are the records of what changed. Do not touch files outside `.claude/user-flows/`.

### Step 12: Confirm

Print a one-paragraph summary including:

- Source domain → target domains
- Total flows moved, tombstones created, tasks moved, references rewritten
- The exact list of file paths created, modified, or moved
- Suggested next step: `/user-flow-dev pending` to confirm flow status didn't drift, or `/user-flow-dev check <new-domain>` if substantial code edits accompanied the split

Listing every file path is non-negotiable.

## Anti-patterns specific to split

- Splitting a domain with <6 flows. Almost always premature.
- Creating a target domain with <3 flows without invoking the distinct-actors / lifecycle / risk exception explicitly with the user.
- Skipping the tombstone step. External references (commit messages, PRs, conversations) need somewhere to land.
- Auto-rewriting tombstone blocks or `Renamed from:` lines. Those are the record of the rename — they must keep the old ID.
- Renumbering flows that didn't change domains.
- Writing files in Phase 1.
