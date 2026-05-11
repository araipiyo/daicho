# Daicho Security Specification

Status: Draft 0.2
Phase: 0

## 1. Purpose

This document defines mandatory security requirements for Daicho specs, runtime code, CLI checks, starter code, and deployment templates.

## 2. Security posture

Daicho must be secure by default. Convenience must not weaken authentication, authorization, tenancy, SQL safety, auditability, secret handling, or supply-chain controls.

## 3. Default project requirements

Default projects must:

- Avoid password login.
- Avoid administrative GUIs.
- Require a principal for requests.
- Deny resource operations unless policy allows them.
- Require every resource to declare tenancy.
- Generate or use tenant-safe query helpers for tenant-scoped resources.
- Use parameterized SQL or an equivalent safe query builder.
- Redact secrets from logs, plans, errors, and manifests.
- Include command-line security checks.

## 4. Authentication

The first identity mode is upstream trusted identity / no-login mode. It may trust identity headers only behind an explicitly configured trusted boundary.

The mode must define:

- Accepted headers.
- Local development simulation.
- Trusted proxy or access-layer assumptions.
- Header-spoofing controls.
- Behavior for absent or malformed identity.

OIDC, SAML, passkeys, service credentials, and passwords are explicit adapters. Password authentication remains opt-in if implemented.

## 5. Authorization

Authorization is deny-by-default. Handlers must call policy checks before mutations and before broad data access.

Policies receive documented inputs and return:

- Decision: allow or deny.
- Safe reason code.
- Optional safe explanation.
- Audit metadata.

Denied requests must not reveal sensitive record existence across tenant boundaries.

## 6. Tenancy

Every resource must use exactly one tenancy mode: `global`, `tenant_scoped`, `tenant_partitioned`, or `system`.

For `tenant_scoped` resources, tenant constraints must apply to reads, lists, updates, deletes, relationship traversal, uniqueness checks, and existing-record lookups.

## 7. SQL safety

Generated and starter code must not interpolate untrusted values into SQL strings. Values use parameter binding. Identifiers use generated or allow-listed safe handling.

Custom SQL escape hatches must be explicit, searchable, tested, reviewed for tenant constraints, and absent from default paths.

## 8. Secrets and redaction

Secrets must not appear in:

- Plans.
- Logs.
- Errors.
- OpenAPI.
- JSON Schema.
- Examples.
- Committed deployment manifests.

If a secret affects review, output a redacted placeholder and safe metadata such as variable name or source name.

## 9. Audit

Applications must write audit events for security-relevant operations. Audit records should include event ID, timestamp, correlation ID, principal ID, tenant ID when applicable, resource, operation, result, safe reason code, and redacted change summary.

Audit persistence failures during mutations must have a specified failure policy. The default should fail closed for security-sensitive mutations.

## 10. Supply chain

Generated or starter projects must include:

- Lockfile enforcement.
- Minimal dependencies.
- Dependency review guidance.
- Vulnerability scanning path.
- SBOM generation path.
- Release signing guidance.
- Template integrity expectations.

Dependencies affecting authentication, authorization, SQL, cryptography, or deployment require heightened review.

## 11. Acceptance

Phase 1 code must prove:

- Missing policies deny access.
- Cross-tenant reads and writes fail.
- SQL injection payloads are parameterized or rejected.
- Secrets are redacted from plans and logs.
- Password login is absent from the default template.
- Audit events are emitted for allowed and denied security-relevant operations.
