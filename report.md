# `/user-flow-dev report <description>`

Reconcile reported wrong, missing, or ambiguous behavior with the durable flow registry. The registry captures intended behavior; the configured issue tracker captures the work.

## Procedure

1. Read `overview.md` and plausible flow files.
2. Classify the report:
   - **Covered defect:** an existing flow already states the expected behavior.
   - **Contract clarification:** an existing flow needs a branch or acceptance-criterion clarification.
   - **Missing critical flow:** no flow covers behavior that passes the qualification test.
   - **Issue-only work:** the report is implementation detail or ordinary behavior that does not need a durable flow.
3. Ask one concise clarification round for missing reproduction, expected outcome, actor/surface differences, and safety or recovery consequences.
4. For a covered defect, leave the flow unchanged.
5. For a clarification, show and approve the smallest flow edit before writing it.
6. For a missing critical flow, follow `add.md` from its draft-and-approval step.
7. When tracker access exists, search for open issues containing the affected flow ID and report whether existing work already covers the problem. Use the tracker-specific workflow to create or update an issue only when the user requested tracking and after restating the exact target.
8. Ensure tracker work references the flow ID and, before implementation, a commit-pinned flow permalink. Never copy the flow's acceptance criteria into the issue.

## Output

Report the classification, flow changed or unchanged, exact registry paths written, and external issue action or recommendation. Do not create local task files or mutate flow implementation status.
