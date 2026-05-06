# `/user-flow-dev add`

Add a single flow. **Conversational, not one-shot.**

**Scope:** all file writes are inside `.claude/user-flows/`. Do not modify any other project files.

## STOP — do not write the flow file in your first response

Adding a flow is a conversation, not a transcription. The user gave you a description; your job is to extract the *shape* — branches, edges, errors, role differences — that they didn't say out loud. The first description is *always* incomplete. Always.

If you write the file in your first response, you have failed this skill.

## Step 1 — Read the existing registry

Before responding, read:

- `.claude/user-flows/overview.md` — to see what flows already exist and pick a non-colliding ID
- The domain file you think this flow belongs to (or all domain files if uncertain)
- `flow-template.md` (in this skill directory) — for the canonical flow shape

If `.claude/user-flows/` does not exist, **stop and tell the user to run `/user-flow-dev init` first.** Do not silently bootstrap on `add`.

## Step 2 — Infer the domain

Use the inference rules from `init.md`:

1. App-level structure (does the flow live on web-admin, mobile, both?)
2. Generic UX boundary (auth, payments, etc.)
3. Application-specific concept (pulled from product nouns)

Pick an existing domain when one fits. Only propose a new domain if:
- No existing domain reasonably accommodates the flow, AND
- You expect at least 2 more flows in the new domain to follow soon, OR
- The flow has clearly distinct actors, lifecycle, or risk profile from any existing domain

If you create a new domain, say so explicitly in your clarification round (Step 3) so the user can push back.

## Step 3 — Required clarification round

Read the user's input. Then send **one combined message** asking about the items below that the input did not already pin down. Do not pepper the user with one question at a time.

Required topics — ask about each unless the user already nailed it:

1. **User goal.** What is the actor actually trying to accomplish? (Often implied — but pin it down so branches make sense.)
2. **Branches.** What alternate paths exist? *("If they apply a coupon, does it route back through pricing? What if the coupon is invalid?")*
3. **Errors.** What can go wrong? Network, validation, permission denied, payment declined, race conditions.
4. **Roles.** Are there differences in what different actors can do? *(Player vs venue admin vs system?)*
5. **Surfaces.** Web, mobile, both, or admin-only?
6. **Preconditions.** What state must exist for this flow to start?
7. **Acceptance criteria.** What is observably true if it works? Aim for 2–4 criteria; observable from outside the system.
8. **Dependencies.** Does this flow require another flow to exist first? (e.g. checkout depends on auth)
9. **Domain placement.** If you're proposing a new domain or unsure, surface that here.

Format the questions as a short numbered list. Reference the user's words where you can, so they can see you understood the seed.

## Step 4 — Wait for the user's answers

Do not proceed until the user answers (or explicitly waives) the items.

If the user pushes back ("just write it, I'll fix it later"), respond once with: "I'll write a draft, but flows that get written hastily get re-written. Two minutes of clarification saves an audit later. Anything you'd skip from this list?" and then comply if they still insist. When a flow is written with waived clarification, set `Status: needs manual validation` (instead of the default `not started`) so it surfaces in `pending` flagged as needing human follow-up — not as ordinary new work.

## Step 5 — Assign the ID

Find the highest existing `UF-<DOMAIN>-NNN` in that domain (in `overview.md`) and increment by 1. Pad to 3 digits. Append-only; never reuse a number even if a flow was deferred.

If the domain doesn't exist yet, the first flow is `UF-<DOMAIN>-001`.

## Step 6 — Write the flow

Two writes, both inside `.claude/user-flows/`:

### 6a. Append to `overview.md`

Add a single line to the matching `### <domain>` section. If the domain section doesn't exist, add it (alphabetical order under `## Flows`) and add the domain link to the `## Domains` list at the top.

Format (must match exactly):

```
UF-<DOMAIN>-NNN | <Title> | <domain> | tags: <tag>, <tag>, <tag>
```

### 6b. Write to the domain file

If the domain file exists:
- Append the same one-liner to the `## Index` section
- Append the full flow detail at the bottom under a new `## UF-<DOMAIN>-NNN — <Title>` header, with a `---` separator before it

If the domain file does not exist, create it with the shape from `init.md` Step 7 (2-sentence summary + index + first flow detail).

Flow detail follows `flow-template.md` exactly — do not invent new sections. Default `Status: not started`; use `Status: needs manual validation` if the user waived clarification (Step 4). The `### Branches / error paths` section must contain at least one entry — a flow with no branches is a sign Step 3 was skipped, go back.

## Step 7 — Confirm

Print:
- The new flow ID
- The domain it landed in (and whether the domain was new)
- The branches captured
- The exact file paths that were modified
- A one-line "next: `/user-flow-dev task <FLOW-ID>` if you want to track implementation"

## Anti-patterns specific to add

- One-shotting the flow from the user's first message.
- Asking questions one at a time — combine them.
- Writing a flow with no branches or no acceptance criteria.
- Putting the flow in a brand-new single-flow domain when a sibling domain fits and the flow has no distinct actor/lifecycle/risk.
