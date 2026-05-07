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

Also accept any project description from `$ARGUMENTS` after the word `init`. On a project that already has code, treat it as additional signal layered on top of the scan. On a greenfield project (see Step 1b), it becomes the primary signal.

### Step 1b: Detect greenfield and clarify

If the Step 1 signal is sparse — no `apps/`, `services/`, `packages/`, or `src/` with non-trivial content; no migrations; `package.json` absent or with no real dependencies; README empty or templated — treat this as a **greenfield** project.

In greenfield mode:

- The project description (from `$ARGUMENTS` and/or what you elicit in this step) is the **primary** signal, not additional.
- If no description was provided in `$ARGUMENTS`, ask once: "Looks like a new project — describe what you're building." Wait for the response before continuing.
- Run **one** broad clarification round before proposing domains. 3–5 questions max, covering:
  1. Target users / actors (admin, regular user, lesser admin, guest, etc.).
  2. Top user-visible features for an MVP.
  3. Anything explicitly out of scope for now.
  4. Scale / maturity expectations (single-tenant prototype vs multi-tenant SaaS, etc.) — only if it would change the domain shape.
  5. Anything the user wants modeled as its own concern even if small (e.g. a billing area planned for later).
- Treat the answers as on-par with code signal — they replace the scan inputs that don't exist yet.

Domain split/merge isn't supported yet. On greenfield, **prefer broader, more general domains**. It is easier to live with a slightly fat domain and refine it manually later than to have flows scattered across thin domains the skill can't currently re-home for you.

If the project is not greenfield, skip this step and continue with Step 2 normally.

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
- **Greenfield:** source 1 (app-level structure) is unavailable; sources 2 and 3, plus the clarification round answers, become primary. Aim for the low end of "4–8 domains" and tolerate fatter ones — domain split/merge tooling doesn't exist yet, so it is cheaper to resplit manually later than to start with thin domains.

### Step 3: Generate candidate flows

For each domain, list 4–10 candidate flows. Each candidate needs:

- Stable ID `UF-<DOMAIN>-NNN`
- Behavioral title (active voice, present tense, actor-first when applicable)
- 2–5 tags (prefer 2–3 sharp ones over 5 fuzzy ones — see `flow-template.md` for the canonical rule)
- Mental note of branches you'd cover, but don't write them yet

Capture branching at this stage by listing it in a comment, not the title. The flow `UF-PAYMENTS-002 — Subscribe via Stripe Checkout` should mentally include "card declined", "user abandons checkout", "webhook arrives late" — they become branch entries in Phase 2.

On greenfield, branches come from the clarification-round answers and obvious edges those answers imply. Don't fabricate branches the user didn't allude to — let them land later via `add` or `report` once the user has a real opinion on them.

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

Each domain file leads with a 2-sentence summary, then the flow index (one pipe-delimited line per flow, same as `overview.md`), then full flow detail separated by `---`. Each flow detail starts with `## UF-<DOMAIN>-NNN — <Title>` (so an agent can grep-jump by ID) and follows `flow-template.md` exactly — read it before writing.

Status assignment depends on mode:
- **Normal init:** set `Status: init` for every flow. The flows describe behavior already in the code; `check` will evaluate them.
- **Greenfield init:** set `Status: not started` for every flow. There is no existing code to "infer from," and `not started` correctly surfaces the flows in `pending` as work to do.

Do not invent sections that aren't in `flow-template.md`. The 2-sentence summary at the top is the same one-clause summary used in `overview.md`'s domains list.

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
- **Before touching code in a domain:** read `.claude/user-flows/domains/<domain>.md`. The two-sentence summary plus the index tells you which flows you might affect. If a flow you're about to touch carries an `Active task:` line, read that task file too — it has the latest findings or in-flight scope.
- **Before making a change that would alter behavior:** check the affected flows' acceptance criteria. If a change would break one, surface it to the user before making the change.
- **When the user reports a bug, regression, or a "should work like X" expectation:** propose `/user-flow-dev report` so the registry stays in sync. The skill will classify (existing flow vs. new flow), ask clarifying questions, edit AC or create the flow, and generate a task. Don't run it autonomously — surface it and let the user agree.
- **Finishing implementation on a task:** when you complete implementation work tied to a `TASK-NNN`, call `/user-flow-dev done TASK-NNN`. This stages the task as `needs manual validation` and flips the linked flow's status to match — it does **not** archive. The task file gets a manual-validation checklist for the user.
- **After the user manually validates:** the user runs `/user-flow-dev validated TASK-NNN` (or asks you to). That archives the task and flips the flow to `completed`. Do not run `validated` autonomously unless the user has explicitly said the validation passed.
- **After meaningful changes in a domain:** run `/user-flow-dev check <domain>` so flow status and tasks stay reconciled with the code. `check` updates statuses, refreshes the `## Findings` section of the relevant task (or creates one if needed), and archives tasks whose flow now passes.
- **New behavior:** when adding meaningful new behavior, run `/user-flow-dev add <description>` so the flow gets documented before or alongside the change.
```

If the user accepts, insert the snippet as a top-level `## User flows` section near other architectural notes (after any "Architecture" section, before "Task Completion Policy" or similar). If CLAUDE.md doesn't exist, ask the user whether to create one with just this section.

If the user declines, do not write to CLAUDE.md and note the decline in the final summary.

### Step 10: Propose `.claude/settings.json` permission rules (preview, then confirm)

`check` and `done` can fire many edits inside `.claude/user-flows/` in one run. Without an allowlist, each edit triggers an approval prompt. Offer to add allow rules to the project's `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Edit(.claude/user-flows/**)",
      "Write(.claude/user-flows/**)"
    ]
  }
}
```

Show the user the addition (merging into any existing `permissions.allow` array if one is present, creating the file otherwise) and ask before writing. If they decline, note it and move on. The skill works fine without it; the rules just remove an interruption.

### Step 11: Confirm

Print a one-paragraph summary including:
- Number of domains and total flows created
- The exact list of file paths written under `.claude/user-flows/`
- Whether CLAUDE.md was updated, declined, or skipped
- Whether the `.claude/settings.json` permission rules were added, declined, or skipped
- Status note (normal init): all generated flows have `Status: init` (the sentinel for "inferred from code, not yet evaluated") so `pending` will be empty until `check` evaluates flows or `add` introduces new ones — this is normal, not a sign the system is empty
- Status note (greenfield init): all generated flows have `Status: not started` and will surface in `pending` immediately as work to do; `check` won't be useful until there's code to verify against
- Suggested next step (normal): `/user-flow-dev check [domain]` to evaluate ACs against current code (this is what flips flows out of `init` status into `completed` / `incomplete` / `issues` / `needs manual validation`), then `/user-flow-dev pending` to see what needs attention
- Suggested next step (greenfield): `/user-flow-dev pending` to see the seeded flows, pick one to implement, then call `/user-flow-dev done <TASK-ID>` when implementation lands and `/user-flow-dev validated <TASK-ID>` after manual verification

Listing every file path is non-negotiable — the user must be able to see what changed without diffing.

## Anti-patterns specific to init

(File-scope and "infer domains" rules live in SKILL.md's Four invariants — not restated here.)

- Phase 1 writing files. Don't.
- Generating fewer than 3 candidate flows for a domain unless that domain has distinct actors/lifecycle/risk that justify keeping it small.
- Generating more than 12 flows for a domain. Split it.
- Adding flows for behavior that the README and code clearly don't support — *unless this is greenfield init*, in which case the project description plus clarification-round answers are the support. Even in greenfield, don't invent behavior the user didn't allude to; let it land later via `add` or `report`.
- Inventing template fields not in `flow-template.md`. The template is canonical.
