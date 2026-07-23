---
name: user-flow-dev
description: Use when a repository has a user-flow registry, a UF-* flow ID appears, work changes a critical user journey, or a user reports or approves a behavioral change that should survive across issues and agent sessions.
---

# User-flow-centered development

Maintain a small registry of durable behavioral contracts. User flows define what users must experience; the repository's issue tracker and project board own implementation scope, dependencies, status, and history.

## Resolve the registry

Before every command:

1. Use the user-flow path declared by repository instructions.
2. Otherwise use the one existing registry among `docs/user-flows/`, `.Codex/user-flows/`, and `.claude/user-flows/`.
3. If more than one exists, stop and ask which is canonical.
4. If none exists, only `init` may create one, at `docs/user-flows/`.

Call the resolved path `FLOW_ROOT`.

## Commands

Read the matching file in full before acting.

| Command | Reference | Purpose |
|---|---|---|
| `init` | `init.md` | Propose and create the first critical flows. |
| `add <description>` | `add.md` | Add one durable behavioral contract. |
| `report <description>` | `report.md` | Reconcile reported behavior with an existing or new flow. |
| `decision <description>` | `decision.md` | Record an approved change to intended behavior. |
| `check [domain or FLOW-ID]` | `check.md` | Verify flow acceptance criteria against code and tests without writing work-tracking state. |

Unknown commands print this table and stop.

## Registry shape

```text
FLOW_ROOT/
  overview.md
  flows/
    UF-<DOMAIN>-NNN.md
```

`overview.md` contains only domain links and one dense line per flow:

```text
UF-<DOMAIN>-NNN | <Title> | <domain> | tags: <tag>, <tag>
```

Each flow has one file and follows `flow-template.md`. IDs are append-only and filenames never change. Superseded flows remain in the index and point to their replacement.

## What deserves a flow

Document behavior only when it is important enough that a future agent must reconstruct it before changing code. A candidate normally has at least two of these properties:

- crosses multiple screens, services, roles, or asynchronous steps;
- carries privacy, authorization, safety, financial, or data-loss risk;
- has meaningful branches or failure recovery;
- encodes a long-lived product decision likely to outlive one issue;
- would cause a costly regression if an agent inferred the behavior from code alone.

Keep ordinary CRUD, isolated UI details, refactors, and technical acceptance criteria in the issue tracker. `init` proposes 4–8 critical flows total, not an exhaustive product catalog.

## Source-of-truth boundaries

| Concern | Canonical owner |
|---|---|
| Intended user behavior, branches, observable acceptance criteria | Flow file |
| Implementation scope and technical acceptance criteria | Issue |
| Priority, dependencies, assignment, and delivery status | Issue tracker / project board |
| Review, validation evidence, and merge state | Pull request |
| Historical flow contract | Flow file at the pinned Git commit |

Never create local task files, task indices, archives, or implementation statuses inside `FLOW_ROOT`.

## Issue and pull-request integration

Flow-to-work relationships are many-to-many. Store them in issues and pull requests, never as mutable issue links inside flow files.

Before implementation of work that affects a documented flow:

1. Read the current flow file.
2. Confirm the issue covers the intended behavior and repository dependency rules.
3. Pin the exact contract in the issue body under `## User-flow contracts`:

   ```markdown
   ## User-flow contracts
   - `UF-ACCESS-006` — [`<short-sha>`](https://github.com/OWNER/REPO/blob/FULL_SHA/docs/user-flows/flows/UF-ACCESS-006.md)
   ```

4. If the flow is new or changed for this work, approve and commit the flow before implementation code, then pin that commit.
5. If the pinned file differs from the current file when work resumes, review the flow diff before coding. Re-scope or split the issue when behavior changed; update the pin only after that reconciliation.
6. Reference the same pinned contracts in the pull request and report acceptance-criterion evidence there.

Use the repository's configured tracker integration and lifecycle guidance for issue reads and writes. This skill never creates a parallel backlog or mirrors tracker status.

## Invariants

- Describe behavior from the actor's perspective, not tables, endpoints, libraries, or components.
- Keep acceptance criteria observable and testable.
- Ask a clarification round before creating or materially changing a flow.
- Keep one behavioral source: issues may reference flow criteria but must not copy them.
- Let Git history version contracts; do not maintain a separate revision counter.
- `check` is evidence-based and read-only. It may recommend tracker work but never creates it silently.

## Common mistakes

- Cataloging every screen during `init`.
- Adding task status, issue URLs, or implementation notes to a flow.
- Copying flow acceptance criteria into an issue.
- Linking an issue only to a mutable branch or current file instead of a commit permalink.
- Treating a changed flow as though an older issue still has the same scope.
- Claiming an acceptance criterion holds without a code or test citation.
