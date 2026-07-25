# Claude Code Project Instructions

Use `agents/WEB_SPEC_COMPLIANCE_VERIFIER.md` for every functional or technical change that affects application behaviour, data, authorization or testability.

Required sequence:

1. `PLAN`
2. implementation
3. targeted tests
4. independent `INSPECT`
5. correction and affected test rerun
6. repeat until `INSPECT_CLEAN`
7. full applicable test campaign
8. actor/data/RLS coverage gates
9. `VERIFY`

Repository stack:

- React + TypeScript + Tailwind CSS
- Supabase
- Vitest + React Testing Library
- MSW
- Playwright
- Supabase CLI + pgTAP
- Deno Test
- axe-core
- Lighthouse CI

Do not modify expected product behaviour without an approved specification decision. Do not run destructive Supabase commands against a remote project. Do not expose service-role secrets. Final approval belongs to VERIFY, not to the implementation loop.
