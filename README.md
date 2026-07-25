# Claude React Supabase Loop Engineering

Autonomous specification-driven test, inspection and correction framework for Claude Code projects built with React, TypeScript, Tailwind CSS and Supabase.

**Version:** 1.0.0  
**License:** Apache-2.0

## Workflow

```text
PLAN
  -> implementation
  -> targeted tests
  -> independent INSPECT
  -> correction
  -> affected test rerun
  -> repeat until INSPECT_CLEAN
  -> full test campaign
  -> VERIFY
  -> GO / GO WITH CONDITIONS / NO-GO
```

A green test suite alone is not proof of specification compliance. The framework requires traceability from requirement to implementation, actor/data state, test, evidence and verdict.

## Supported testing stack

- Vitest
- React Testing Library
- MSW
- Playwright E2E and visual testing
- Supabase CLI and pgTAP
- Deno Test for Edge Functions
- axe-core
- Lighthouse CI

## Claude Code package

```text
CLAUDE.md
.claude/settings.json
.claude/agents/spec-verifier.md
.claude/agents/test-fix-loop.md
agents/WEB_SPEC_COMPLIANCE_VERIFIER.md
workflows/CLAUDE_LOOP_ENGINEERING_WORKFLOW.md
templates/TEST_ACTOR_MANIFEST_TEMPLATE.md
docs/INSTALL.md
```

The `test-fix-loop` agent implements and corrects code. The independent `spec-verifier` agent audits each iteration from a fresh context.

## Safety

The default Claude permissions allow local tests and local Supabase reset operations, but block Git pushes, destructive Git cleanup, remote Supabase linking, database pushes, migration repair, secret changes and Edge Function deployment.

Never expose `SUPABASE_SERVICE_ROLE_KEY` to browser code, test traces, screenshots or committed fixtures.

## Installation

Read [`docs/INSTALL.md`](docs/INSTALL.md), copy the framework files into the target repository and map the commands to the scripts that actually exist in its `package.json`.

Commands that are unavailable must be recorded as `NOT_CONFIGURED` or `BLOCKED`; Claude must never invent successful execution.
