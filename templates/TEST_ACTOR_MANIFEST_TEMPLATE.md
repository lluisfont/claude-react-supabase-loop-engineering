# Test Actor Manifest Template

**Version:** 1.0.0

Use this manifest to plan, validate, reserve, reset and reconcile test actors without exposing secrets.

## Rules

- One row represents one verifiable combination of actor, role, permissions, tenant, auth state and business-data state.
- Never store passwords, JWTs, OTP values, service-role keys, private keys or production personal data.
- Use stable actor references and secure secret-source references.
- Shared actors must be reserved to avoid concurrent E2E interference.
- Readiness requires validated authentication, effective permissions, RLS state and deterministic reset.

## Actor inventory

| ACTOR_REF | Use case IDs | Role/claims | Tenant | Auth state | Business-data state | Secret source | Preparation | Reset | Reservation | Login validated | RLS validated | Result |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `qa.admin.tenant-a` | `UC-001` | `admin` | `tenant-a` | Active | Empty account | `local-env/QA_ADMIN_PASSWORD` | Seed script | DB reset | Exclusive E2E | No | No | `NOT_VALIDATED` |

## Readiness results

- `READY`
- `NOT_VALIDATED`
- `MISSING_TEST_ACTOR`
- `INVALID_ROLE_OR_PERMISSION`
- `INVALID_TENANT`
- `INVALID_BUSINESS_STATE`
- `CREDENTIAL_EXPIRED_OR_LOCKED`
- `NON_RESETTABLE_TEST_DATA`
- `NON_DETERMINISTIC_SHARED_USER`
- `BLOCKED_BY_EXTERNAL_IDENTITY_PROVIDER`
- `NOT_APPLICABLE`

## Test execution reconciliation

| Test ID | Use case ID | ACTOR_REF | Expected role/state | Actual role/state | Fixture version | Reset performed | RLS case | Evidence | Result |
|---|---|---|---|---|---|---|---|---|---|

## Security review

- [ ] No credentials, tokens or service-role keys are present.
- [ ] No production personal data is present.
- [ ] Secret sources are references only.
- [ ] Shared-account reservation is documented.
- [ ] Reset and RLS validation procedures are documented.
