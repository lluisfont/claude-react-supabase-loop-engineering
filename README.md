# Claude React Supabase Loop Engineering

A reusable **specification-driven implementation, testing, inspection and correction framework** for Claude Code projects built with:

- React
- TypeScript
- Tailwind CSS
- Supabase

The repository gives Claude Code a controlled engineering process in which it can implement a change, run the appropriate tests, inspect the result independently, correct verified defects and repeat until the code is ready for final verification.

**Version:** 1.0.0  
**License:** Apache-2.0

---

## Why this repository exists

AI coding agents are very effective at producing code, but a successful build or a green test suite does not prove that the implementation matches the approved requirement.

Typical failures include:

- implementing only the happy path;
- changing expected behaviour to make a test pass;
- writing tests that reproduce the implementation instead of the specification;
- using mocks that bypass the real database or authorization rules;
- declaring a feature complete without validating RLS, tenant isolation or test data;
- fixing one layer while breaking another;
- repeatedly changing code without measuring whether risk is actually decreasing.

This framework replaces the uncontrolled cycle of:

```text
prompt -> code -> tests pass -> done
```

with a traceable engineering process:

```text
requirement
  -> scenario
  -> implementation
  -> actor and data state
  -> appropriate test layer
  -> evidence
  -> independent verdict
```

The goal is not merely to obtain green tests. The goal is to produce evidence that the software implements the approved specification safely and reproducibly.

---

## Core workflow

```text
Approved specification
        |
        v
      PLAN
        |
        v
  Implementation
        |
        v
 Targeted tests
        |
        v
Independent INSPECT
        |
        +----------------------+
        | blocking defect?     |
        |                      |
        +---- yes -------------+
        |                      |
        v                      |
    Correction                 |
        |                      |
        v                      |
Affected tests rerun ----------+
        |
        v
  INSPECT_CLEAN
        |
        v
Full applicable test campaign
        |
        v
      VERIFY
        |
        v
GO / GO WITH CONDITIONS / NO-GO
```

The implementation loop ends at `INSPECT_CLEAN`. That state means the implementation is ready for the complete verification campaign. It is **not** release approval.

Final acceptance belongs to `VERIFY`.

---

## The logic of the framework

### 1. PLAN

Before modifying code, the verifier reads the approved specification and creates:

- stable requirement and scenario IDs;
- requirement-to-code expectations;
- the required test type for each scenario;
- actor, role, permission and authentication requirements;
- Supabase data fixtures and reset requirements;
- positive, unauthorized and cross-tenant RLS cases;
- expected evidence;
- known ambiguities, dependencies and blockers.

Typical output:

```text
docs/qa/<change-name>/01-verification-plan.md
```

PLAN does not change application code and does not emit a release verdict.

### 2. Implementation

The `test-fix-loop` agent implements only behaviour that is traceable to an approved requirement or to a verified inspection finding.

It must not silently:

- expand scope;
- redesign authentication or authorization;
- approve destructive migrations;
- change visual baselines;
- weaken valid assertions;
- disable RLS;
- skip failing tests;
- redefine the expected product behaviour.

### 3. Targeted testing

After each coherent change, Claude runs the smallest valid test set for the affected layer.

Examples:

| Change | Minimum expected validation |
|---|---|
| Pure function, validator, reducer or transformation | TypeScript check + Vitest |
| React component or hook | Vitest + React Testing Library |
| Frontend network behaviour | React Testing Library + MSW |
| Routing, login or browser persistence | Playwright |
| Tailwind layout or responsive behaviour | Playwright + visual comparison when configured |
| Migration, trigger, constraint or RLS policy | Local Supabase reset + pgTAP |
| Edge Function | Deno Test + integration or E2E when required |
| Accessibility-sensitive UI | axe-core + keyboard and focus review |
| Performance-sensitive change | Production build + Lighthouse CI |

A mock is not accepted as proof of behaviour that belongs to another layer. For example, MSW can validate frontend error handling, but it cannot prove that an RLS policy prevents cross-tenant access.

### 4. Independent INSPECT

The `spec-verifier` agent reviews the change from a fresh, read-only context.

It compares:

- the approved specification;
- the Git diff;
- the implementation;
- tests and fixtures;
- Supabase migrations and policies;
- available execution evidence.

It classifies findings such as:

- `CODE_DEFECT`
- `MISSING_IMPLEMENTATION`
- `SPEC_GAP`
- `SPEC_AMBIGUITY`
- `DESIGN_DEFECT`
- `TESTABILITY_GAP`
- `OUT_OF_SCOPE_CHANGE`
- `SECURITY_OR_DATA_RISK`

The implementation agent does not approve its own work. A correction is only considered resolved after a fresh INSPECT confirms it.

### 5. Correction loop

When INSPECT identifies a valid defect, the implementation agent:

1. fixes the smallest coherent cause;
2. reruns the affected checks;
3. records the commands and evidence;
4. requests another independent inspection.

The loop continues until one of these outcomes is reached:

- `INSPECT_CLEAN`
- `BLOCK_FOR_SPEC_DECISION`
- `BLOCKED_BY_DEPENDENCY`
- `SCOPE_DRIFT`
- `STAGNATED`
- `OSCILLATING`
- `REPEATED_FAILURE`
- `BUDGET_EXHAUSTED`
- `SAFETY_STOP`
- `LOOP_ESCALATION_REQUIRED`
- `ABORTED`

The framework is deliberately bounded. It does not allow Claude to keep changing code indefinitely.

### 6. Full test campaign

After `INSPECT_CLEAN`, Claude runs every applicable test layer defined in PLAN. Commands that do not exist or cannot be executed must be recorded as:

- `NOT_CONFIGURED`
- `NOT_RUN`
- `BLOCKED`

Claude must never invent successful execution.

### 7. VERIFY

VERIFY reconciles each approved scenario with:

- the actual implementation;
- the test that ran;
- the actor and role used;
- the fixture and database state;
- the RLS state;
- the browser or environment;
- the evidence generated.

It emits one of three verdicts:

- `GO`
- `GO WITH CONDITIONS`
- `NO-GO`

---

## Claude Code agents

### `test-fix-loop`

Path:

```text
.claude/agents/test-fix-loop.md
```

Responsibilities:

- implement approved requirements;
- choose the smallest valid test set;
- correct verified defects;
- preserve scope;
- record each iteration;
- stop at `INSPECT_CLEAN` or a controlled terminal state.

This agent can edit code, tests, fixtures, migrations and non-secret configuration within the approved scope.

### `spec-verifier`

Path:

```text
.claude/agents/spec-verifier.md
```

Responsibilities:

- create PLAN;
- inspect the implementation independently;
- map requirements to code and tests;
- review React, TypeScript and Tailwind behaviour;
- review Supabase migrations, Auth, RLS, Storage, Realtime and Edge Functions;
- classify findings and severity;
- perform final VERIFY.

This agent is configured as a read-only auditor.

---

## Supported testing stack

| Layer | Recommended tool | What it proves |
|---|---|---|
| Unit and domain logic | Vitest | Deterministic rules, transformations, validators and error handling |
| React components | React Testing Library | Rendering and behaviour through user-visible interactions |
| Frontend API boundary | MSW | Loading, success, empty, malformed, delayed and error responses |
| Browser flows | Playwright | Routing, authentication, authorization and real user journeys |
| Visual regression | Playwright screenshots | Stable layout and intentional visual changes |
| Database and RLS | Supabase CLI + pgTAP | Constraints, functions, triggers, permissions and tenant isolation |
| Edge Functions | Deno Test | Input validation, auth decisions, errors, retries and idempotency |
| Accessibility | axe-core | Automated accessibility violations, complemented by manual focus checks |
| Performance | Lighthouse CI | Explicit performance and quality budgets |

The framework does not require every project to use every tool. PLAN determines which layers are applicable to each change.

---

## Repository structure

```text
claude-react-supabase-loop-engineering/
├── README.md
├── CLAUDE.md
├── CHANGELOG.md
├── LICENSE
├── agents/
│   └── WEB_SPEC_COMPLIANCE_VERIFIER.md
├── workflows/
│   └── CLAUDE_LOOP_ENGINEERING_WORKFLOW.md
├── templates/
│   └── TEST_ACTOR_MANIFEST_TEMPLATE.md
├── docs/
│   └── INSTALL.md
├── examples/
│   ├── CLAUDE.example.md
│   ├── loop-config.example.md
│   └── verification-context.example.md
├── .claude/
│   ├── settings.json
│   └── agents/
│       ├── spec-verifier.md
│       └── test-fix-loop.md
└── .github/
    └── pull_request_template.md
```

### File responsibilities

| File | Purpose |
|---|---|
| `CLAUDE.md` | Main operating rules loaded by Claude Code in the target project |
| `agents/WEB_SPEC_COMPLIANCE_VERIFIER.md` | Detailed PLAN, INSPECT and VERIFY policy |
| `workflows/CLAUDE_LOOP_ENGINEERING_WORKFLOW.md` | Iteration, stopping, progress and safety rules |
| `.claude/agents/test-fix-loop.md` | Editable implementation and correction agent |
| `.claude/agents/spec-verifier.md` | Independent read-only verification agent |
| `.claude/settings.json` | Safe default command permissions and denials |
| `templates/TEST_ACTOR_MANIFEST_TEMPLATE.md` | Test users, roles, data states, resets and evidence |
| `examples/verification-context.example.md` | Per-change environment and execution context |
| `.github/pull_request_template.md` | Pull request compliance checklist |

---

# Installation guide

## 1. Prerequisites

The target application should have:

- Claude Code installed and authenticated;
- Node.js supported by the application;
- npm, pnpm, yarn or the package manager already used by the project;
- React and TypeScript;
- Tailwind CSS when applicable;
- Supabase CLI when the project uses Supabase;
- a local container runtime compatible with the Supabase CLI;
- Deno when Supabase Edge Functions are present.

Do not change the application's package manager merely to install this framework.

## 2. Clone this repository

Clone it outside the target application or into a temporary directory:

```bash
git clone --depth 1 https://github.com/lluisfont/claude-react-supabase-loop-engineering.git
```

This repository is an installation package. It is not intended to replace the target application's repository.

## 3. Copy the framework files

Copy these files and directories into the root of the target application:

```text
CLAUDE.md
agents/WEB_SPEC_COMPLIANCE_VERIFIER.md
workflows/CLAUDE_LOOP_ENGINEERING_WORKFLOW.md
templates/TEST_ACTOR_MANIFEST_TEMPLATE.md
.claude/settings.json
.claude/agents/spec-verifier.md
.claude/agents/test-fix-loop.md
.github/pull_request_template.md          optional
examples/                                  optional
```

### Bash example

Run from the target application directory and replace `<framework-path>`:

```bash
mkdir -p agents workflows templates .claude/agents .github
cp <framework-path>/CLAUDE.md ./CLAUDE.md
cp <framework-path>/agents/WEB_SPEC_COMPLIANCE_VERIFIER.md ./agents/
cp <framework-path>/workflows/CLAUDE_LOOP_ENGINEERING_WORKFLOW.md ./workflows/
cp <framework-path>/templates/TEST_ACTOR_MANIFEST_TEMPLATE.md ./templates/
cp <framework-path>/.claude/settings.json ./.claude/settings.json
cp <framework-path>/.claude/agents/spec-verifier.md ./.claude/agents/
cp <framework-path>/.claude/agents/test-fix-loop.md ./.claude/agents/
cp <framework-path>/.github/pull_request_template.md ./.github/
```

### PowerShell example

Run from the target application directory and replace `<framework-path>`:

```powershell
New-Item -ItemType Directory -Force agents, workflows, templates, .claude/agents, .github | Out-Null
Copy-Item <framework-path>/CLAUDE.md ./CLAUDE.md
Copy-Item <framework-path>/agents/WEB_SPEC_COMPLIANCE_VERIFIER.md ./agents/
Copy-Item <framework-path>/workflows/CLAUDE_LOOP_ENGINEERING_WORKFLOW.md ./workflows/
Copy-Item <framework-path>/templates/TEST_ACTOR_MANIFEST_TEMPLATE.md ./templates/
Copy-Item <framework-path>/.claude/settings.json ./.claude/settings.json
Copy-Item <framework-path>/.claude/agents/spec-verifier.md ./.claude/agents/
Copy-Item <framework-path>/.claude/agents/test-fix-loop.md ./.claude/agents/
Copy-Item <framework-path>/.github/pull_request_template.md ./.github/
```

### Existing Claude configuration

Do not overwrite an existing `CLAUDE.md`, `.claude/settings.json` or pull request template blindly.

When these files already exist:

1. merge the Loop Engineering rules into the existing instructions;
2. preserve project-specific conventions;
3. review command permissions individually;
4. keep the deny rules for remote or destructive operations;
5. verify every referenced path after the merge.

## 4. Install the applicable testing dependencies

Install only the tools required by the project.

Example with npm:

```bash
npm install --save-dev \
  vitest \
  jsdom \
  @testing-library/react \
  @testing-library/user-event \
  @testing-library/jest-dom \
  msw \
  @playwright/test \
  axe-core \
  @axe-core/playwright \
  @lhci/cli
```

For a project using a locally versioned Supabase CLI, add it according to the project's package-management policy. Edge Functions use Deno's test runner.

After installing Playwright, install the browsers required by the project:

```bash
npx playwright install
```

## 5. Map the real project commands

Inspect the target application's existing `package.json`. Add or adapt scripts without duplicating commands that already exist.

Example:

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

This is an example, not a mandatory script layout. Update `CLAUDE.md` and `.claude/settings.json` so they reference the commands that really exist.

## 6. Configure local Supabase testing

Destructive database tests must use a local Supabase environment.

Typical commands are:

```bash
supabase start
supabase db reset
supabase test db
```

The installation is not ready until:

- all migrations rebuild a clean local database;
- seed data is deterministic;
- tests can reset their state;
- pgTAP covers material constraints, functions and RLS policies;
- positive and negative authorization cases exist;
- tenant isolation is tested where applicable;
- no production project is linked for autonomous destructive execution.

Never allow the loop to automatically run:

```text
supabase link
supabase db push
supabase migration repair
supabase secrets set
supabase functions deploy
```

## 7. Review Claude permissions

The supplied `.claude/settings.json` permits common local inspection and test commands while denying operations such as:

- `git push`;
- `git reset --hard`;
- destructive Git cleanup;
- remote Supabase linking;
- remote database pushes;
- migration repair;
- secret updates;
- Edge Function deployment.

Review the allow list for the actual package manager and scripts used by the application. Remove permissions that are unnecessary.

## 8. Create the verification context

For each feature, fix or change, create:

```text
docs/qa/<change-name>/00-verification-context.md
```

Use [`examples/verification-context.example.md`](examples/verification-context.example.md) as the starting point.

Minimum information:

```text
CHANGE_NAME=
SPEC_PATH=
BASE_REF=main
TARGET_REF=
NODE_VERSION=
PACKAGE_MANAGER=
REACT_VERSION=
TYPESCRIPT_VERSION=
SUPABASE_CLI_VERSION=
BROWSERS=chromium
TEST_ENVIRONMENT=local
LOOP_POLICY=optional
```

Never store passwords, JWTs, refresh tokens, OTP values, service-role keys or production data in this file.

## 9. Validate the installation

Before using the loop on a real feature, confirm:

```text
[ ] Claude Code loads the root CLAUDE.md
[ ] spec-verifier appears as an available agent
[ ] test-fix-loop appears as an available agent
[ ] package scripts referenced by CLAUDE.md exist
[ ] TypeScript and lint commands execute
[ ] Vitest executes at least one known test
[ ] Playwright can launch the configured browser
[ ] local Supabase starts successfully
[ ] supabase db reset rebuilds the database
[ ] pgTAP tests can execute
[ ] Edge Function tests execute when applicable
[ ] no secret is committed or exposed to browser code
[ ] remote destructive commands remain denied
```

---

## First execution

### Create PLAN

Use Claude Code with a prompt such as:

```text
Use the spec-verifier agent in PLAN mode.
Read CLAUDE.md and the approved specification at <spec-path>.
Create docs/qa/<change-name>/01-verification-plan.md.
Include requirements, scenarios, actors, fixtures, RLS cases, required test layers, evidence and blockers.
Do not modify application code.
```

### Start the implementation loop

```text
Use the test-fix-loop agent for <change-name>.
Follow the approved verification plan and CLAUDE_LOOP_ENGINEERING_WORKFLOW.md.
Implement only approved scope.
Run the smallest valid tests after each correction.
Require a fresh independent INSPECT after every implementation change.
Stop at INSPECT_CLEAN or a controlled terminal state.
```

### Run final verification

After `INSPECT_CLEAN` and the complete applicable test campaign:

```text
Use a fresh spec-verifier agent in VERIFY mode.
Reconcile every requirement with the actual implementation, command output, actor, fixture, data and RLS state, browser and evidence.
Create docs/qa/<change-name>/03-final-verification-report.md.
Emit GO, GO WITH CONDITIONS or NO-GO.
```

---

## Evidence structure

```text
docs/qa/<change-name>/
├── 00-verification-context.md
├── 01-verification-plan.md
├── 02-implementation-inspection.md
├── test-actor-manifest.md
├── evidence/
│   ├── command-results.md
│   ├── screenshots/
│   ├── playwright/
│   ├── database/
│   └── accessibility/
├── loop/
│   ├── 00-loop-context.md
│   ├── iteration-01/
│   ├── iteration-02/
│   └── loop-closure.md
└── 03-final-verification-report.md
```

Every iteration should record:

- Git hash and refs;
- requirements or findings addressed;
- files changed;
- commands executed;
- results and evidence paths;
- actor and data state;
- database reset and RLS state;
- risk score before and after;
- remaining blockers;
- continuation or stop decision.

---

## Recommended loop policy

Default repository policy:

```text
LOOP_POLICY=optional
LOOP_AUTO_REQUIRE_ON_HIGH_RISK=true
LOOP_MAX_ITERATIONS=5
LOOP_MAX_STAGNANT_ITERATIONS=2
LOOP_MAX_REPEAT_FAILURES=2
LOOP_MAX_TRANSIENT_RETRIES=3
```

Use `required` for changes involving:

- authentication or authorization;
- RLS policies;
- multi-tenant data isolation;
- database migrations;
- destructive or irreversible operations;
- security controls;
- Edge Functions with side effects;
- payment or billing flows;
- broad refactors;
- P0 or P1 remediation;
- large AI-generated diffs.

---

## What this framework does not do

It does not:

- guarantee the complete absence of defects;
- replace product decisions or an approved specification;
- replace professional security or database review;
- approve production deployment;
- merge or push code automatically;
- authorize destructive remote database operations;
- turn every test into an E2E test;
- treat coverage percentage as proof of correctness;
- allow the implementation agent to approve its own work.

---

## Further documentation

- [Installation and usage details](docs/INSTALL.md)
- [Web Spec Compliance Verifier](agents/WEB_SPEC_COMPLIANCE_VERIFIER.md)
- [Claude Loop Engineering Workflow](workflows/CLAUDE_LOOP_ENGINEERING_WORKFLOW.md)
- [Test Actor Manifest Template](templates/TEST_ACTOR_MANIFEST_TEMPLATE.md)
- [Example Claude instructions](examples/CLAUDE.example.md)
- [Example verification context](examples/verification-context.example.md)
- [Changelog](CHANGELOG.md)

---

## Security rules

Never commit:

- passwords;
- JWTs or refresh tokens;
- OTP values;
- private keys;
- Supabase service-role keys;
- production personal data;
- unrestricted test accounts shared without reservation and reset controls.

Use references instead:

```text
ACTOR_REF=qa.admin.tenant-a
SECRET_SOURCE=local-env/SUPABASE_QA_ADMIN_PASSWORD
```

The Supabase service-role key must never be exposed to browser code, Playwright traces, screenshots or committed fixtures.

---

## License

Licensed under the Apache License 2.0. See [LICENSE](LICENSE).

## Disclaimer

This framework improves traceability and autonomous correction discipline. It does not guarantee the absence of defects and does not replace professional security review, database review or human acceptance testing.
