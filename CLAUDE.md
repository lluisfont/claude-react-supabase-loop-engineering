# Claude Code Repository Instructions

## Mission

Implement approved specifications for this React, TypeScript, Tailwind CSS and Supabase repository through a controlled test-fix-inspect loop. Do not declare completion merely because code compiles or tests pass.

## Sources of truth

Read in this order:

1. this file and repository instructions;
2. approved specification, issue or acceptance criteria;
3. design documents, database contracts and ADRs;
4. implementation tasks;
5. current code and configuration;
6. test evidence.

When expected behaviour is missing or ambiguous, stop the affected requirement with `BLOCK_FOR_SPEC_DECISION`. Do not invent product behaviour.

## Required workflow

1. Run `PLAN` using `agents/WEB_SPEC_COMPLIANCE_VERIFIER.md`.
2. Implement only approved in-scope behaviour.
3. Run the smallest relevant test set.
4. Run `INSPECT` using a fresh verifier context.
5. Fix only verified code, test, fixture or configuration defects.
6. Repeat until `INSPECT_CLEAN` or a controlled terminal state.
7. Run the complete applicable test campaign.
8. Run actor, test-data and RLS coverage gates.
9. Run `VERIFY` and produce the final verdict.

Use `workflows/CLAUDE_LOOP_ENGINEERING_WORKFLOW.md` when Loop Engineering is active.

## Technology-specific test mapping

- Pure functions, hooks, reducers, validators and transformations: Vitest.
- React rendering and interaction: React Testing Library with user-event.
- Network-dependent frontend behaviour: MSW with explicit success, empty, malformed and error responses.
- Critical browser flows: Playwright.
- Layout-sensitive behaviour: Playwright visual comparisons with controlled viewport, fonts and data.
- Postgres functions, constraints, triggers and RLS: Supabase CLI plus pgTAP.
- Edge Functions: Deno Test, with external services mocked or isolated.
- Accessibility: axe-core plus keyboard/focus checks.
- Performance budgets: Lighthouse CI.

Do not replace a database/RLS test with an MSW mock. Do not claim browser behaviour from a component test. Do not use snapshots as the sole assertion for business behaviour.

## Default command discovery

Before execution, inspect `package.json`, lockfile, Supabase config and existing scripts. Prefer repository scripts. Typical commands are:

```bash
npm ci
npm run typecheck
npm run lint
npm run test -- --run
npm run test:components
npm run test:e2e
npm run test:visual
npm run test:a11y
npm run lighthouse
supabase start
supabase db reset
supabase test db
deno test supabase/functions --allow-env
npm run build
```

Never claim a command ran when it did not. Record missing scripts as `NOT_CONFIGURED`.

## Autonomous correction permissions

Claude may modify source code, tests, fixtures, migrations and non-secret configuration only when the change is directly traceable to an approved requirement or an INSPECT finding.

Claude must stop for:

- ambiguous or missing product behaviour;
- destructive or irreversible migration decisions;
- production data access;
- secrets or credentials;
- authentication/authorization redesign outside approved scope;
- material dependency or architecture changes;
- scope expansion;
- deployment, merge or release approval.

## Supabase safety

- Use local Supabase for destructive tests.
- Never run destructive commands against a remote project.
- Never expose `SUPABASE_SERVICE_ROLE_KEY` to client code.
- Verify both positive and negative RLS cases.
- Verify cross-tenant isolation where tenancy exists.
- Migrations must be forward reproducible from a clean database.
- Seed and reset operations must be deterministic.
- Edge Functions must validate authentication and authorization independently of the client.

## Evidence

Store reports under `docs/qa/<change-name>/`. Every iteration must record changed files, findings addressed, commands, results, remaining blockers and the Git hash.

## Completion rule

The implementation loop ends at `INSPECT_CLEAN`. Release acceptance requires a separate `VERIFY` result of `GO` or explicitly accepted `GO WITH CONDITIONS`.
