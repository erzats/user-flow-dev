# Flow template

Canonical structure for a single flow's detail section. Used by `init` and `add` when writing flows into a domain file. The same template applies to every flow regardless of domain.

## Template

```markdown
## UF-<DOMAIN>-NNN — <Title>

Status: <not started | init | incomplete | issues | needs manual validation | completed | deferred | superseded by UF-X-NNN>
Active task: tasks/<domain>/TASK-NNN.md   ← optional; present only while a task is open for this flow
Surface: <web | mobile | both | admin-only>
Actor: <role(s) — e.g. "player", "venue admin", "system">
Tags: <tag>, <tag>, <tag>

### User goal
<One sentence: why does the actor want to do this? The behavioral motivation, not the implementation.>

### Preconditions
- <What must be true for the flow to start. Often references other flows ("user is signed in via UF-AUTH-001") or system state.>

### Main path
1. <Step from the actor's perspective.>
2. ...

### Branches / error paths
- **<Branch name — happy alternate>:** <How this path differs from main. Numbered sub-steps if non-trivial.>
- **<Error name — what goes wrong>:** <What the actor sees and what state results.>

### Acceptance criteria
- <Observable statement that must hold for the flow to be considered correct.>
- <Another observable.>

### Depends on
<UF-... IDs of flows that must exist for this one to make sense, or "none">
```

## Field rules

- **Status:** Required, one of: `init`, `not started`, `incomplete`, `issues`, `needs manual validation`, `completed`, `deferred`, `superseded by UF-X-NNN`. `init` is the sentinel for "inferred from code, not yet evaluated by check" and is what the `init` command writes. `add` defaults new flows to `not started`. `check` writes `incomplete` / `issues` / `needs manual validation` / `completed` based on per-AC verdicts. Humans set `deferred` and `superseded` (intent signals `check` cannot infer). `pending` surfaces only `not started` / `incomplete` / `issues` / `needs manual validation`; the others are considered settled. See SKILL.md "Status values" for the full mapping.
- **Active task:** Optional. Present only while a task file exists for this flow. Added by `task` (manual) or `check` (findings). Removed by `done` or by `check` when the linked task is archived. Format is the relative path from the domain file: `tasks/<domain>/TASK-NNN.md`. The flow detail should never carry a stale `Active task:` line — at most one task per flow, and the line is the canonical pointer to it.
- **Surface:** `web`, `mobile`, `both`, or `admin-only`. Used by `check` to narrow the search area in code.
- **Actor:** role of whoever drives the flow. `system` is acceptable for background flows (webhook reconciliation, scheduled jobs).
- **Tags:** 2–5 short keywords, lowercase and hyphenated. These are the same tags that appear in the `overview.md` index line for this flow — keep them in sync.
- **User goal:** one sentence, behavioral. NOT "implements X" or "calls Y". The actor's actual aim — e.g. "subscribe so the club stays active beyond the trial".
- **Preconditions:** what must be true outside this flow for it to start. Often references other flow IDs.
- **Main path:** the happy path, numbered. Each step is something observable from the actor's perspective. Avoid implementation verbs ("system calls X", "API returns Y") unless the actor's view depends on it.
- **Branches / error paths:** bullet list. Each entry is `**<Name>:**` followed by a description or sub-steps. Always include at least one — a flow with only a main path is incomplete.
- **Acceptance criteria:** 2–4 observable statements. Avoid implementation language (no table names, no function names, no library calls). "User lands on home" is observable; "session token is written to AsyncStorage" is not.
- **Depends on:** flow IDs only. If none, write `none`.

## Common mistakes

- Writing a flow with no branches. Real behavior has branches — go back and ask.
- Writing acceptance criteria that reference internal state. Rewrite as observable behavior.
- Putting implementation hints in the steps. The flow is what the user sees, not what the system does internally.
- Skipping the User goal because the title "says it all". The title states the what; the User goal states the why. Without the why, branches are easy to miss.
- Padding tags. 2–3 sharp tags beat 5 fuzzy ones. `tags: stripe, subscription, checkout` not `tags: stripe, payment, subscription, billing, checkout, recurring, money`.
