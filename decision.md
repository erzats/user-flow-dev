# `/user-flow-dev decision <description>`

Record a user-approved product decision that deliberately changes an existing flow. Use this when the user says the documented behavior should evolve — not when they report a defect.

## Procedure

1. Resolve `FLOW_ROOT` using `SKILL.md`, then read `FLOW_ROOT/overview.md` and the affected domain file.
2. Identify the existing flow. If the decision may require a new flow, use `add` instead.
3. Ask only for material gaps. Confirm actor, surface, intended behavior, and important branches; do not repeat facts already established in the conversation.
4. Make the smallest behavioral edit to the flow: user goal, main path, branches, and observable acceptance criteria as needed. Explain any rewritten AC in the confirmation.
5. Keep the flow's status unchanged. A product decision is not automatically an `issues` report. If an open task exists, add a concise decision note to its `## Scope` or `## Notes`; never rewrite `## Findings`.
6. Do not create a task unless unimplemented behavior remains and the user asks to track it.

## Confirm

Print:

```
Decision recorded.
- Flow: UF-<DOMAIN>-NNN
- Behavioral change: <one sentence>
- Status: unchanged (<status>)
- Task: <updated existing task | none>
```

## Guardrails

- Do not use `decision` to hide a regression; use `report` when current behavior violates an accepted criterion.
- Do not mark an acceptance criterion verified merely because it was rewritten. Use `check` after implementation or meaningful code changes.
