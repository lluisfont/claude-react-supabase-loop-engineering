# Claude Code Loop Engineering Workflow

**Version:** 1.0.0  
**Depends on:** `agents/WEB_SPEC_COMPLIANCE_VERIFIER.md`

## Objective

Repeatedly implement, test, inspect and correct approved behaviour until `INSPECT_CLEAN` or a controlled stop condition.

```text
PLAN -> implement -> targeted tests -> independent INSPECT -> correction -> affected tests -> fresh INSPECT
```

The loop does not emit release approval. Final approval belongs to VERIFY after the complete applicable test campaign.

## Activation

```text
LOOP_POLICY=disabled|optional|required
LOOP_POLICY_OVERRIDE=disabled|optional|required
LOOP_MAX_ITERATIONS=5
LOOP_MAX_STAGNANT_ITERATIONS=2
LOOP_MAX_REPEAT_FAILURES=2
LOOP_MAX_TRANSIENT_RETRIES=3
```

Require the loop for auth, RLS, migrations, tenant isolation, destructive/data-sensitive behaviour, Edge Functions with side effects, security changes, broad refactors and P0/P1 remediation.

## Iteration procedure

1. Load the specification, PLAN and latest checkpoint.
2. Select only open approved findings/requirements.
3. Implement the smallest coherent correction.
4. Run formatting/type/lint checks relevant to changed files.
5. Run the smallest valid affected test set.
6. Record commands, changed files and results.
7. Start a fresh independent `spec-verifier` review.
8. Classify all findings and calculate progress.
9. Persist a checkpoint.
10. Continue, close, block, escalate or abort.

Do not hide a failing test by weakening assertions, deleting coverage, changing expected behaviour, over-mocking the subject under test, disabling RLS or skipping tests.

## Layer-aware correction order

1. Specification decision, when genuinely required.
2. Database schema/RLS contract.
3. Edge Function/API contract.
4. frontend domain/state logic.
5. React UI behaviour.
6. tests and fixtures, only when incorrect or incomplete.
7. documentation/evidence.

When a lower layer changes, rerun all dependent upper-layer tests.

## Mandatory test selection

- Type/logic changes: typecheck + Vitest.
- Component changes: Vitest/RTL; use MSW for network boundaries.
- Routing/auth/browser persistence: Playwright.
- CSS/layout changes: targeted Playwright plus visual tests when configured.
- Schema, trigger or policy changes: clean `supabase db reset` + pgTAP.
- Edge Functions: Deno Test plus integration/E2E where behaviour crosses boundaries.
- Accessibility changes: axe-core plus keyboard/focus verification.
- performance-sensitive changes: build + Lighthouse CI.

## Safety stops

Stop autonomous changes for:

- `SPEC_AMBIGUITY` affecting expected behaviour;
- destructive migration or remote database operation;
- production data or secrets;
- new authentication/authorization architecture;
- intentional visual baseline approval;
- major dependency/framework upgrade;
- deployment, merge or release decision;
- scope drift.

## Progress

```text
RISK_SCORE = P0*100 + P1*25 + P2*5 + P3
```

Material progress means fewer P0/P1 findings, lower risk score, more traced requirements, resolved blockers or new valid evidence. Code churn is not progress.

Return `STAGNATED` after the configured number of iterations without material progress. Return `OSCILLATING` when fixes repeatedly reopen prior findings. Return `REPEATED_FAILURE` when the same deterministic failure recurs without a material change.

## Iteration record

```text
Iteration:
Timestamp:
Base ref:
Target ref:
Git hash:
Specification version:
Files changed:
Requirements/findings addressed:
Commands and results:
Database reset/migration result:
RLS/actor state used:
Corrections made:
INSPECT result:
Remaining blockers:
Risk score before/after:
Circuit breaker state:
Decision:
```

Allowed decisions: `CONTINUE`, `BLOCK_FOR_SPEC_DECISION`, `BLOCK_FOR_INFRASTRUCTURE`, `BLOCK_FOR_TEST_ACTOR`, `ESCALATE`, `INSPECT_CLEAN`, `ABORT`.

## INSPECT CLEAN

Requires concrete implementation evidence for all in-scope requirements; no contradiction; no open P0/P1; no material RLS, auth, secret or data risk; no scope drift; reproducible migrations; actor/data planning aligned with code; applicable targeted tests passing; and no blocker in the fresh INSPECT report.

Any subsequent code, migration, policy, dependency, fixture or configuration change invalidates `INSPECT_CLEAN` and requires fresh inspection.

## Terminal states

- `INSPECT_CLEAN`
- `BUDGET_EXHAUSTED`
- `STAGNATED`
- `OSCILLATING`
- `REPEATED_FAILURE`
- `SAFETY_STOP`
- `BLOCK_FOR_SPEC_DECISION`
- `SCOPE_DRIFT`
- `BLOCKED_BY_DEPENDENCY`
- `BLOCK_FOR_TEST_ACTOR`
- `CONTEXT_RESET_REQUIRED`
- `LOOP_ESCALATION_REQUIRED`
- `ABORTED`

Every terminal report must state reason, evidence, residual risk, persisted state, required decision, owner and resume condition.
