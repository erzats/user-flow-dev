# `/user-flow-dev add <description>`

Add one critical behavioral contract. Never write it from the user's first description.

## Procedure

1. Read `overview.md`, all plausible sibling flow files, and `flow-template.md`.
2. Apply the qualification test from `SKILL.md`.
   - If the behavior is ordinary implementation scope, explain that it belongs in the issue and stop.
   - If an existing flow already covers it, surface that flow and use `decision` or `report` instead.
3. Ask one concise clarification round covering only missing material facts: actor and goal, preconditions, main outcome, important role/branch/failure differences, observable acceptance criteria, and dependencies.
4. Wait for the answers. If the user waives clarification, produce a draft for review but do not write it until they approve the draft.
5. Assign the next append-only ID in the inferred domain.
6. Show the complete proposed flow and obtain approval.
7. Append its dense index line to `overview.md` and create `flows/UF-<DOMAIN>-NNN.md` from `flow-template.md`.
8. Report the exact paths changed and remind the user that implementation issues must pin the committed flow file rather than copy its acceptance criteria.

## Guardrails

- One flow per invocation.
- Keep one flow per file.
- Never create a task, issue, status, or tracker mirror as a side effect.
- Do not add a flow merely because a new issue exists.
