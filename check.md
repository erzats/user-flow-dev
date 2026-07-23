# `/user-flow-dev check [domain or FLOW-ID]`

Verify acceptance criteria against current code and tests. This command is strictly read-only: it reports evidence and never updates flow files or work-tracking state.

## Scope

- `check UF-<DOMAIN>-NNN` checks one flow.
- `check <domain>` checks every indexed flow in that domain.
- `check` checks every flow and warns about the expected cost before starting.

## Procedure

1. Read `overview.md` and only the selected flow files.
2. Use surface, tags, actor, and flow language to locate relevant code and tests.
3. Evaluate each acceptance criterion:

   | Verdict | Requirement |
   |---|---|
   | `✓ holds` | Cite the specific file and line/function or test that enforces it. |
   | `⚠ manual` | Cite the implementation location and explain why runtime or human validation is still required. |
   | `✗ broken` | Cite code or a failing test that contradicts it. |
   | `– not implemented` | No corresponding implementation exists after targeted search. |

4. Never award `✓` from naming, comments, or apparent intent. Use `⚠ manual` only after locating relevant implementation. If targeted search finds no implementation, use `– not implemented`.
5. When tracker access exists, search open issues and pull requests for the flow ID. Report coverage, stale pinned-contract links, or missing tracking; do not create or edit tracker items silently.
6. Print evidence per acceptance criterion and a summary by verdict.

## Output

```text
## UF-ACCESS-006 — Approved person enters a parish
- ✓ Approved email gains one membership — path/file.ts:42-70
- ⚠ Cross-device email link — callback exists at app/auth/callback/route.ts:10-26, but cross-device behavior requires a runtime test
- ✗ No redundant acceptance step — app/onboarding/page.tsx:40 still renders Accept

Related work:
- Issue #16 pins <sha>; current flow differs — scope reconciliation required

Summary: 1 holds, 1 manual, 1 broken.
```

Do not write statuses, findings files, task records, issue comments, or project-board changes.
