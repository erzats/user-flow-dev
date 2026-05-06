# `/user-flow-dev report <description>`

Capture a user-reported issue, regression, or behavior wish and route it to the right place in the registry — either updating an existing flow or proposing a new one. **Conversational. Never one-shot.**

`report` is the entry point for in-conversation feedback like:
- "X is broken on mobile" → likely affects an existing flow.
- "When I do Y, Z should happen" → either tightens AC on an existing flow or describes a new one.
- "We never specced this but it matters" → likely a missing flow.

**Scope:** writes inside `.claude/user-flows/` only.

## When Claude should call this autonomously

When the user describes user-visible behavior that's wrong, missing, or ambiguous — and they're not already in another flow-skill subcommand — propose `report` before doing anything else. Examples that should trigger:

- "It's broken when I…", "doesn't work for…", "I expected X but got Y"
- "It should also handle…", "we need to support…"
- "I don't think we ever defined what happens when…"
- A bug discovered mid-implementation that affects user-visible behavior beyond the current task

If the user is already mid-task or asking for a code change, you don't have to interrupt with `report` for every aside — but if the aside describes a regression or unspecified behavior, surface it: "That sounds like it touches UF-X-NNN. Want me to run `/user-flow-dev report` on it?"

Do not call `report` autonomously without telling the user. It's conversational by design.

## Procedure

### Step 1 — Classify

Read the report against `overview.md` plus any obviously-relevant domain files. Decide one of:

- **A. Affects an existing flow.** The behavior described is covered (or should be covered) by a documented flow. The work is to modify that flow's AC or branches, then generate or update a task.
- **B. Describes a flow that should exist but doesn't.** No documented flow covers it. The work is to add one, then generate a task.
- **C. Out of scope.** The report isn't a behavior issue (it's a refactor request, a question, infrastructure work, etc.). Tell the user this skill isn't the right home and stop.

If you're not sure between A and B, ask the user before proceeding. Don't guess.

### Step 2 — Clarification round (always conversational)

Whether A or B, gather enough to write or edit the flow correctly. Ask **2–4 focused questions**, not a wall. Ground each question in something concrete:

- What's the user's surface? (Web vs. mobile vs. both. If A, confirm against the existing flow's surface.)
- What's the expected behavior in the specific case? (One sentence. Avoid "should work correctly".)
- What's the current behavior? (For A: which AC is being violated. For B: what happens today.)
- Are there branches you've already thought about? (Empty state, error path, edge case.)

If the user already gave you most of this in the initial description, ask only for the gaps. If they said "it's broken when X" without a repro, ask for the repro.

Stop the clarification round when you can write the AC and branches without guessing. If you're still guessing, ask one more question.

### Step 3a — Path A: modify an existing flow

Open the linked flow's domain file. Make the smallest edit that captures the report:

- **New AC** — append a bullet to `### Acceptance criteria` describing the user's expectation. Use observable language ("the page renders the loading state for at least 200ms before showing the empty state"), not implementation language.
- **New branch** — append a bullet to `### Branches / error paths` if the report describes an unhandled case.
- **AC clarification** — rewrite an existing AC bullet only if the report shows the current bullet is ambiguous in a way that caused the issue. Note the change in your confirmation output. Don't silently rewrite.

Set the flow's `Status: issues` (since the report describes a real-world failure of an AC). If a task already exists for this flow, update it; if not, create one with `**Source:** report` (see Step 4 task shape).

### Step 3b — Path B: create a new flow

Use the same procedure as `add.md` (read it). Key differences from `add`:

- The trigger is a user report, not a proactive "let's spec something". The behavior often already half-exists in code, so you may need to skim the codebase to write AC honestly.
- Set `Status: issues` if the report is a regression of half-implemented behavior, `not started` if it's net-new behavior.
- The task that follows should have `**Source:** report`.

Run through `add`'s clarification + write steps, then come back here for Step 4.

### Step 4 — Generate or update a task

For Path A with an open task → update its `## Scope` section (or `## Notes` if scope already covers it) to reference the report. Do not rewrite findings; that's `check`'s job.

For Path A with no open task, or Path B → create a task using the same shape as `task.md`, but with:

```markdown
**Source:** report
**Reported:** <YYYY-MM-DD>
**Reporter:** user
```

Pick a TASK ID per the rules in `task.md` Step 3. Append to `todo.md`. Add `Active task:` to the flow detail.

In the `## Scope` of the task, capture *what the user said* — verbatim or close to it — so the implementer has the original report alongside the AC. Two-sentence repro plus the expected vs actual outcome is the right shape. Don't restate flow content.

### Step 5 — Confirm

Print:

```
Report routed.
- Path: A (modified UF-<DOMAIN>-NNN) | B (new flow UF-<DOMAIN>-NNN)
- AC change: <summary, or "no AC change — only added a branch" or "no AC change — task captures the wording fix">
- Status: <flow's new status>
- Task: TASK-NNN created (or TASK-NNN updated)

Next: /user-flow-dev check <domain> when the fix lands, or assign TASK-NNN to a session.
```

## Anti-patterns specific to report

- One-shot. The whole point is the clarification round. If you can answer all the questions yourself, you're guessing.
- Adding an AC that just says "X works correctly". An AC names what *correct* means in observable terms, not the absence of failure.
- Silently rewriting an existing AC. If the report shows an AC was wrong, call it out in the confirmation; don't bury the change.
- Creating a flow when an existing one already covers the behavior. Search `overview.md` first.
- Creating a task without ever writing the AC change. The AC is the durable record; the task is the work.
- Calling `report` autonomously without telling the user it's happening. The skill exists to clarify, not to capture-and-go.
- Treating "report" as a synonym for "log this somewhere". If the user is genuinely venting, not reporting a behavioral issue, ask before invoking.
