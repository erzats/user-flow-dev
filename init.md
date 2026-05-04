# `/user-flow-dev init`

First-time setup for a project. Infers domains, proposes flows, gets user confirmation, writes files inside `.claude/user-flows/`, and previews a CLAUDE.md addition.

**Two-phase: propose, then write.** Never write files in the first response.

**Scope:** all file writes during init are inside `.claude/user-flows/` except the CLAUDE.md update, which is always preview-then-confirm. Do not modify or migrate any other files.

## Phase 1 — Propose (no file writes yet)

### Step 1: Gather signal

Read these without asking the user:

- `README.md`, `README`, `docs/README*` — describes what the product does
- `CLAUDE.md` (root and any nested) — describes architecture and conventions
- `package.json` (and any monorepo `apps/*/package.json`) — names of apps
- Top-level folder structure: `apps/`, `packages/`, `services/`, `src/`
- `docs/` — any existing flow-like content
- `supabase/migrations/` or equivalent — table names hint at domains

Also accept any project description from `$ARGUMENTS` after the word `init`. Treat it as additional signal, not a replacement for the codebase scan.

### Step 2: Infer domains

Domains come from three sources, in order of preference:

1. **App-level structure** — `apps/web-admin/` → likely `admin-dashboard` domain. `apps/mobile-reader/` → `mobile-reader`. Names should match folders where reasonable.
2. **Generic UX boundaries** — `auth`, `payments`, `onboarding`, `notifications`, `search`, `settings`. Add only if the README or migrations clearly indicate the area exists.
3. **Application-specific concepts** — pulled from product nouns in the README. "scavenger hunt builder" → `hunt-builder`. "discussion threads" → `discussions`.

Rules:
- Lowercase, hyphenated. `admin-dashboard`, not `AdminDashboard` or `admin_dashboard`.
- Default minimum: 3 flows per domain. If a candidate domain has 1–2 flows, fold them into the nearest neighbor.
- Exception: keep a small domain when it has distinct actors, distinct lifecycle, or distinct risk profile. (See SKILL.md "Domain inference" for the trade-off.)
- Aim for 4–8 domains for a typical small/medium project. More than 10 is usually over-segmented.

### Step 3: Generate candidate flows

For each domain, list 4–10 candidate flows. Each candidate needs:

- Stable ID `UF-<DOMAIN>-NNN`
- Behavioral title (active voice, present tense, actor-first when applicable)
- 2–5 tags (prefer 2–3 sharp ones over 5 fuzzy ones — see `flow-template.md` for the canonical rule)
- Mental note of branches you'd cover, but don't write them yet

Capture branching at this stage by listing it in a comment, not the title. The flow `UF-PAYMENTS-002 — Subscribe via Stripe Checkout` should mentally include "card declined", "user abandons checkout", "webhook arrives late" — they become branch entries in Phase 2.

### Step 4: Present for confirmation

Output the proposal as:

```
Proposed domains and flows:

auth/
  UF-AUTH-001 | Sign in with SSO | auth | tags: sso, session, oauth
  UF-AUTH-002 | First-time account provisioning | auth | tags: signup, sso
  ...

payments/
  UF-PAYMENTS-001 | Subscribe via Stripe Checkout | payments | tags: stripe, subscription, checkout
  ...

[N domains, M flows total]

Confirm, edit, or merge before I write files. Suggestions:
- Add: a flow I missed?
- Drop: a flow that's not real?
- Merge: any thin domains?
- Rename: any domain or flow?
```

**Stop and wait for user response.** Do not write files.

## Phase 2 — Write (after user confirmation)

### Step 5: Create the directory tree

Use Write to create:

```
.claude/user-flows/overview.md
.claude/user-flows/domains/<each-domain>.md
.claude/user-flows/tasks/todo.md
```

The `tasks/<domain>/` and `tasks/archive/` subfolders are created lazily — the first task creation makes its domain folder.

All writes are confined to `.claude/user-flows/`. Do not touch anything else in this step.

### Step 6: Write `overview.md`

Exact format:

```markdown
# User Flows — Overview

Index of all documented flows. Scan here first; load the domain file only when you need detail. Format per line:

`UF-<DOMAIN>-NNN | <Title> | <domain> | tags: <tag>, <tag>`

Conventions: IDs are append-only — never renumber. Superseded flows stay listed (status in domain file). New flows go at the bottom of their domain section.

## Domains

- [auth](domains/auth.md) — <one-clause summary>
- [payments](domains/payments.md) — <one-clause summary>
- ...

## Flows

### auth
UF-AUTH-001 | Sign in with SSO | auth | tags: sso, session, oauth
UF-AUTH-002 | First-time account provisioning | auth | tags: signup, sso
...

### payments
UF-PAYMENTS-001 | Subscribe via Stripe Checkout | payments | tags: stripe, subscription, checkout
...
```

The domain summaries are the same one-clause summaries that lead the domain file (Step 7). Do not write paragraphs.

### Step 7: Write each domain file

Each domain file leads with a 2-sentence summary, then the flow index, then full flow detail. Each flow detail follows `flow-template.md` exactly. Read `flow-template.md` before writing.

Shape:

```markdown
# Domain: <name>

<Two sentences max. What this domain covers and any cross-cutting note that applies to all flows in it (e.g. "SSO only — no password flows" or "All charges go through Stripe").>

## Index

UF-<DOMAIN>-001 | <Title> | <domain> | tags: ...
UF-<DOMAIN>-002 | <Title> | <domain> | tags: ...
...

---

## UF-<DOMAIN>-001 — <Title>

Status: init
Surface: <web | mobile | both | admin-only>
Actor: <role(s)>
Tags: <tag>, <tag>, <tag>

### User goal
<one sentence>

### Preconditions
- <state required>

### Main path
1. ...

### Branches / error paths
- **<Name>:** ...

### Acceptance criteria
- ...

### Depends on
<UF-... IDs or "none">

---

## UF-<DOMAIN>-002 — <Title>
...
```

Rules:
- The `## Index` section repeats the one-liner per flow (yes, the same line that's in `overview.md`). This makes the file scannable on its own.
- Each flow detail starts with `## UF-<DOMAIN>-NNN — <Title>` so an agent can grep-jump to a specific flow by ID.
- `---` separators between flow detail sections.
- Flow content follows `flow-template.md` exactly. Do not invent new sections.

### Step 8: Write `tasks/todo.md`

```markdown
# Tasks

One-line-per-task index. Full detail lives in `tasks/<domain>/TASK-NNN.md`. Done tasks move to `tasks/archive/`.

Format: `TASK-NNN | <summary> | <domain> | <FLOW-ID> | <status>`

(no tasks yet)
```

### Step 9: Update CLAUDE.md (preview, then confirm)

Read the project's `CLAUDE.md`. Show the user this snippet and **ask before adding it** — never silent:

```markdown
## User flows

This project documents intended product behavior under `.claude/user-flows/`. Treat it as canonical for "how this is supposed to work".

- **Session start:** read `.claude/user-flows/overview.md`. It is short and pipe-delimited; load it always.
- **Before touching code in a domain:** read `.claude/user-flows/domains/<domain>.md`. The two-sentence summary plus the index tells you which flows you might affect.
- **Before making a change that would alter behavior:** check the affected flows' acceptance criteria. If a change would break one, surface it to the user before making the change.
- **Finishing a task:** when you complete implementation work tied to a `TASK-NNN`, call `/user-flow-dev done TASK-NNN` so the task file moves to archive and `todo.md` reflects reality.
- **New behavior:** when adding meaningful new behavior, run `/user-flow-dev add <description>` so the flow gets documented before or alongside the change.
```

If the user accepts, insert the snippet as a top-level `## User flows` section near other architectural notes (after any "Architecture" section, before "Task Completion Policy" or similar). If CLAUDE.md doesn't exist, ask the user whether to create one with just this section.

If the user declines, do not write to CLAUDE.md and note the decline in the final summary.

### Step 10: Confirm

Print a one-paragraph summary including:
- Number of domains and total flows created
- The exact list of file paths written under `.claude/user-flows/`
- Whether CLAUDE.md was updated, declined, or skipped
- Note that all generated flows have `Status: init` (the sentinel for "inferred from code, not yet evaluated") so `pending` will be empty until `check` evaluates flows or `add` introduces new ones
- Suggested next step: `/user-flow-dev check [domain]` to evaluate ACs against current code (this is what flips flows out of `init` status into `completed` / `incomplete` / `issues` / `needs manual validation`), then `/user-flow-dev pending` to see what needs attention

Listing every file path is non-negotiable — the user must be able to see what changed without diffing.

## Anti-patterns specific to init

- Phase 1 writing files. Don't.
- Asking the user to name domains. Infer them.
- Asking "should I use this folder structure?". The answer is yes.
- Writing files outside `.claude/user-flows/` during Phase 2 (CLAUDE.md is the only exception, and only after preview-and-confirm).
- Generating fewer than 3 candidate flows for a domain unless that domain has distinct actors/lifecycle/risk that justify keeping it small.
- Generating more than 12 flows for a domain. Split it.
- Adding flows for behavior that the README and code clearly don't support. Better to stop at "main flows" and let the user add later than to fabricate.
- Inventing template fields not in `flow-template.md`. The template is canonical.
