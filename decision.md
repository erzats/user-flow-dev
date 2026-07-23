# `/user-flow-dev decision <description>`

Record an approved product decision that changes a current behavioral contract.

## Procedure

1. Read `overview.md` and the affected flow file.
2. Confirm the decision affects an existing flow. Use `add` if it is a distinct journey.
3. Ask only for missing material facts about actor-visible behavior, branches, failures, or acceptance criteria.
4. Show the exact behavioral diff in plain language and obtain approval.
5. Make the smallest edit to the flow file. Update the overview line only if title or tags changed.
6. Report the changed file and identify open issues or pull requests that reference the flow ID when tracker access is available.
7. Warn that pinned older contracts remain historically valid. Reconcile each active issue's scope before updating its commit permalink.

Do not change work-tracking state, copy implementation notes into the flow, or update issue pins automatically.
