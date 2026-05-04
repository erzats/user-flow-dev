# `/user-flow-dev check [domain]`

Verify documented acceptance criteria still hold against the current code. Flag regressions. Write each examined flow's `Status:` field based on the per-flow verdict.

**Status-write only.** `check` updates the `Status:` line of each examined flow and nothing else. It never modifies AC text, branches, dependencies, or any other field. It never creates tasks.

## STOP — no vibes-based verification

The whole point of `check` is to be the verification loop the rest of the system relies on. If you skim a file, see something that "looks roughly right", and mark an AC as `✓ holds`, you have actively destroyed the value of this skill. The user's project will silently rot while the report says everything is fine.

**An acceptance criterion is verified only if you traced specific code (or a specific test) that demonstrably enforces it.** If you cannot trace it in 60 seconds of focused search, the verdict is `⚠ unclear`, never `✓ holds`. There is no in-between.

The cost of a false positive (claiming a broken flow works) is much higher than a false negative (flagging a working flow as unclear). Calibrate accordingly.

## Scope

- `check` (no arg) — checks every active flow across all domains. Slow. Use before a release. Warn the user about runtime up front.
- `check <domain>` — checks only flows in one domain. Use before merging a feature branch that touched that area.

Skip flows whose status is `deferred` or `superseded by ...`. Those encode product/refactoring intent that `check` has no signal for and must not overwrite. Examine all other flows: `init`, `not started`, `incomplete`, `issues`, `needs manual validation`, `completed`. For these statuses, `check` will rewrite the value based on the new verdict.

## Procedure

### Step 1 — Load the target flows

If a domain was specified:
- Read `.claude/user-flows/domains/<domain>.md` only.
- If the domain file doesn't exist, list available domains from `overview.md` and stop.

If no domain was specified:
- Read `overview.md` to enumerate domains.
- Then read each `.claude/user-flows/domains/*.md` in turn.
- Warn the user: "Checking N flows across M domains; this will take a while."

For each flow, extract:
- Flow ID and title
- Surface (web/mobile/both/admin-only) — narrows where to search
- Acceptance criteria (the bullet list under `### Acceptance criteria`)
- Tags — additional search hints

### Step 2 — Locate the implementation

For each flow, find the area that implements it. Use:

- The flow's surface to pick the right app folder (`apps/web/`, `apps/mobile/`, both)
- The domain name as a search hint
- Tags as additional search hints
- The flow title's nouns and verbs

Use Grep and Read. For a thorough sweep across a large area, dispatch the Explore agent. Do NOT guess — if you cannot locate the implementation with a few targeted searches, the verdict for that flow's ACs is `– not implemented` or `⚠ unclear` (whichever fits). Inability to find code is itself useful signal.

### Step 3 — Evaluate each acceptance criterion

For each AC bullet, decide one of:

| Verdict | Meaning | When to use |
|---|---|---|
| `✓ holds` | Code or tests demonstrably enforce the criterion. **Required:** cite a file path and a function or line range. | Only when you traced specific code that enforces the AC. No exceptions. |
| `⚠ unclear` | Implementation exists but you can't tell from static reading whether the AC holds. | When tracing was inconclusive — async behavior, runtime config, untested branches. Default to this when in doubt. |
| `✗ broken` | Code or tests demonstrably violate the criterion. | When you can point to specific code that contradicts the AC. Cite where and explain. |
| `– not implemented` | The flow's implementation isn't present at all. | When no code corresponds to the flow. Distinct from `unclear`. |

A `✓ holds` without a file:line citation is invalid — downgrade it to `⚠ unclear`. A `⚠` you can't resolve in 60 seconds is a `⚠`, not a `✓`.

### Step 4 — Print the report

Format:

```
## Check report — <scope>

### auth/UF-AUTH-001 — Sign in with SSO
- ✓ Sign-in completes in under 5 seconds — apps/web/src/app/auth/route.ts:42-78
- ✓ Cancelling OAuth never produces an error toast — apps/mobile/screens/SignIn.tsx:114
- ⚠ Session persists across app restarts — needs runtime test; AsyncStorage logic at apps/mobile/contexts/AuthContext.tsx:60 looks correct but expiry handling is unclear

### payments/UF-PAYMENTS-002 — Subscribe via Stripe Checkout
- ✓ App state never says "subscribed" before Stripe confirms — apps/web/src/app/api/stripe-webhook/route.ts:30
- ✗ Successful payment converges to "subscribed" within 5 minutes — no reconciliation job exists. Webhook is the only path; if dropped, state never converges. (apps/web/src/app/api/stripe-webhook/route.ts handles only the webhook)

[continued for each flow]

---

## Summary

Flows checked: 18
- 14 fully passing (every AC ✓)
- 2 with unclear ACs (manual verification needed)
- 1 with broken ACs (UF-PAYMENTS-002)
- 1 with no implementation (UF-NOTIF-005)

### Action items
1. UF-PAYMENTS-002: add a reconciliation job for missed webhooks.
2. UF-NOTIF-005: not implemented; consider running `/user-flow-dev task UF-NOTIF-005` to track.
```

### Step 5 — Write status updates

For each examined flow, compute the per-flow verdict and rewrite its `Status:` line in the domain file. Status mapping (highest priority wins):

| Per-flow condition | New status |
|---|---|
| Any AC `✗ broken` | `issues` |
| Any AC `⚠ unclear` (and none broken) | `needs manual validation` |
| Mix of `✓ holds` and `– not implemented` (no broken, no unclear) | `incomplete` |
| All ACs `– not implemented` | `not started` |
| All ACs `✓ holds` | `completed` |

Edit only the `Status:` line. Do not touch anything else in the flow's detail section.

If the new status equals the existing status, you may still rewrite (idempotent) or skip — both are fine.

If the flow had `Status: deferred` or `Status: superseded by ...`, you should not be in this step at all (those flows are skipped at Step 1).

### Step 6 — Do not modify anything else

`check` writes the `Status:` field and nothing else.

- Do not auto-create tasks for broken flows. Suggest `/user-flow-dev task <FLOW-ID>` in the action items; let the user run it.
- Do not edit AC text, branches, preconditions, or any other field. Those change via deliberate edits, not as a side effect of check.
- Do not stamp a "last checked" timestamp anywhere.
- Do not flip `deferred` or `superseded` flows to anything else — those are off-limits.

## Anti-patterns specific to check

- **Vibes-based verification.** Marking ACs `✓ holds` because the code "looks roughly right". If you didn't trace specific code, it's `⚠`. This is the most important rule in this file.
- Reasoning about ACs against the AC text alone, without searching the codebase. The check is against code, not against itself.
- Citing a file without a line range or function. A bare filename is not a citation.
- Modifying any flow field other than `Status:`. ACs and other content change deliberately, not as a side effect.
- Touching `deferred` or `superseded` flows.
- Auto-creating tasks for findings. The user decides which findings become tasks.
- Treating `check` (no arg) as quick. It is not. Warn up front.
