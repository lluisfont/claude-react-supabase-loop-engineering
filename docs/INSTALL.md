# Installation and Usage

**Version:** 1.0.0

## Prerequisites

- Claude Code installed and authenticated.
- React and TypeScript project.
- Node.js and the repository's package manager.
- Supabase CLI for local database tests.
- Applicable testing tools installed.

Typical development dependencies:

```bash
npm install -D vitest @testing-library/react @testing-library/user-event @testing-library/jest-dom jsdom msw @playwright/test axe-core @axe-core/playwright @lhci/cli
```

Supabase Edge Functions use Deno Test. Database and RLS tests use Supabase CLI plus pgTAP.

## Copy these files

```text
CLAUDE.md
agents/WEB_SPEC_COMPLIANCE_VERIFIER.md
workflows/CLAUDE_LOOP_ENGINEERING_WORKFLOW.md
.claude/settings.json
.claude/agents/spec-verifier.md
.claude/agents/test-fix-loop.md
templates/TEST_ACTOR_MANIFEST_TEMPLATE.md
```

Review `.claude/settings.json` before use and remove permissions for commands not used by the project.

## Configure repository commands

Inspect `package.json` and map scripts to the actual project. A typical setup may include:

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "test:components": "vitest run",
    "test:e2e": "playwright test",
    "test:visual": "playwright test --grep @visual",
    "test:a11y": "playwright test --grep @a11y",
    "lighthouse": "lhci autorun"
  }
}
```

Do not overwrite existing conventions blindly.

## Supabase local setup

Use local Supabase for destructive and resettable testing:

```bash
supabase start
supabase db reset
supabase test db
```

Requirements:

- migrations rebuild a clean database;
- seeds are deterministic;
- pgTAP covers constraints, functions and RLS;
- autonomous runs are not linked to production;
- credentials are referenced through local environment variables.

Claude must not automatically run `supabase db push`, remote migration repair, remote secret changes or Edge Function deployment.

## Create verification context

Create `docs/qa/<change-name>/00-verification-context.md` using `examples/verification-context.example.md`.

## Execute

### PLAN

```text
Use the spec-verifier agent in PLAN mode.
Read <spec-path> and CLAUDE.md.
Create docs/qa/<change-name>/01-verification-plan.md.
Do not modify code.
```

### Loop

```text
Use the test-fix-loop agent for <change-name>.
Follow the approved PLAN and CLAUDE_LOOP_ENGINEERING_WORKFLOW.md.
Run targeted tests after each correction and require fresh independent INSPECT.
Stop only at INSPECT_CLEAN or a controlled terminal state.
```

### Full verification

Run every applicable configured layer:

```bash
npm run typecheck
npm run lint
npm run test
npm run test:e2e
npm run test:visual
npm run test:a11y
npm run lighthouse
supabase db reset
supabase test db
deno test supabase/functions --allow-env
npm run build
```

Record unavailable commands explicitly.

### VERIFY

```text
Use a fresh spec-verifier agent in VERIFY mode.
Reconcile every requirement with actual tests, actor, fixture, RLS state, browser/environment and evidence.
Create docs/qa/<change-name>/03-final-verification-report.md.
```

Installation is complete when both agents load, commands are mapped, local Supabase reset is reproducible, Playwright fixtures are deterministic and secret-handling rules are enforced.
