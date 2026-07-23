# Flow template

Each critical flow lives in `FLOW_ROOT/flows/UF-<DOMAIN>-NNN.md`.

```markdown
# UF-<DOMAIN>-NNN — <Title>

Domain: <lowercase domain>
Surface: <web | mobile | both | admin-only | system>
Actor: <role or system>
Tags: <tag>, <tag>
Superseded by: <UF-X-NNN> <!-- optional; omit while current -->

## User goal

<One sentence describing why the actor uses this flow.>

## Preconditions

- <Observable state required before the flow begins.>

## Main path

1. <Actor-visible step.>
2. <Actor-visible result.>

## Branches and failures

- **<Branch>:** <Alternate path and resulting state.>
- **<Failure>:** <What the actor observes and what remains safe or recoverable.>

## Acceptance criteria

- **AC1:** <Observable, testable outcome.>
- **AC2:** <Observable privacy, recovery, or role boundary when relevant.>

## Behavioral prerequisites

<Flow IDs, or `none`.>
```

## Rules

- Use 2–5 tags and keep the overview index in sync.
- Include at least one meaningful branch or failure path.
- Keep 2–5 acceptance criteria with stable sequential keys (`AC1`, `AC2`, ...); use implementation detail only in issues and tests.
- List only prerequisite user flows under `Behavioral prerequisites`. Issue blockers, delivery order, and technical dependencies belong in the tracker.
- Keep the ID and filename forever. To replace behavior, create a new flow and set `Superseded by`.
- Do not add implementation status, issue links, task references, code locations, or revision numbers.
