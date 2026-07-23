# `/user-flow-dev init`

Create the first small set of critical behavioral contracts. This command is propose-first: never write files before the user confirms the candidate flows.

## Phase 1: propose

### 1. Gather signal

Read repository instructions, README and product docs, application routes, schema/domain names, tests, recent commits, and—when connected—the issue backlog and project hierarchy. Backlog items are product signal, not flow records.

### 2. Select critical journeys

Apply the qualification test in `SKILL.md`. Prefer cross-cutting authorization, privacy, money, destructive actions, multi-role behavior, asynchronous recovery, and other journeys where code alone is an unsafe source of intent.

Propose **4–8 flows total** for the first registry. Do not inventory every screen, CRUD action, or planned feature. Thin domains are acceptable; grouping must not manufacture extra flows.

Assign append-only IDs `UF-<DOMAIN>-NNN`, starting at `001` per domain.

### 3. Present for confirmation

Use this format:

```text
Proposed critical user flows:

access/
  UF-ACCESS-001 | Approved person enters a parish | access | tags: admission, authentication

privacy/
  UF-PRIVACY-001 | Private space remains undiscoverable | privacy | tags: non-disclosure, authorization

N flows across M domains.

These are behavioral contracts, not backlog items. Confirm, remove, merge, or rename before I write files.
```

Stop and wait.

## Phase 2: write after confirmation

### 4. Create the registry

If no canonical path is declared, create:

```text
docs/user-flows/
  overview.md
  flows/
    UF-<DOMAIN>-NNN.md
```

Write `overview.md` as a compact index:

```markdown
# User flows

Critical behavioral contracts that must be read before related implementation work. Work tracking belongs in the repository's issue tracker.

## Flows

### access
UF-ACCESS-001 | Approved person enters a parish | access | tags: admission, authentication
```

Write one file per confirmed flow using `flow-template.md`. Infer branches and acceptance criteria from confirmed product decisions; do not invent policy. If a material branch remains unknown, ask before writing that flow.

### 5. Preview repository instructions

When the repository has an instructions file, preview this adapted block and ask before inserting it:

```markdown
## User flows

Critical behavioral contracts live in `docs/user-flows/`.

- Read `docs/user-flows/overview.md` at session start and the relevant `flows/UF-*.md` files before changing documented behavior.
- Keep work tracking in the configured issue tracker and project board; never create a parallel task registry under `docs/user-flows/`.
- Before implementation, pin each affected flow file at an exact Git commit in the issue's `## User-flow contracts` section.
- If a pinned flow changed, review the diff and reconcile issue scope before coding.
- Reference the same pinned contracts and acceptance-criterion evidence in the pull request.
- Run `/user-flow-dev check <domain-or-ID>` after meaningful changes. It is read-only and evidence-based.
```

### 6. Confirm

List every created or modified path. State the number of flows, that no task/status files were created, whether repository instructions were updated, and the recommended next flow to check or implementation issue to link.

## Guardrails

- Do not exceed eight initial flows without a second explicit confirmation.
- Do not create `tasks/`, `todo.md`, archives, statuses, or issue mirrors.
- Do not mark inferred behavior as verified; `check` provides fresh evidence later.
