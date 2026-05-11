# Daicho Security Specification

Status: Draft 0.1  
Phase: 0

## 1. Purpose

This document defines mandatory security requirements for Daicho specifications, generators, generated applications, and deployment templates.

## 2. Security posture

Daicho must be secure by default. Convenience features must not weaken default authentication, authorization, tenancy, SQL safety, auditability, or supply-chain controls.

## 3. Default project security requirements

Default generated projects must:

- Disable password login.
- Require an explicit identity mode.
- Deny resource operations unless a policy allows them.
- Require every resource to declare a tenancy mode.
- Generate tenant-safe queries for tenant-scoped resources.
- Use parameterized SQL or an equivalent safe query builder.
- Avoid administrative GUIs by default.
- Redact secrets from logs, plans, errors, and generated manifests.
- Include command-line security checks.

## 4. Authentication

The first supported identity mode should be upstream trusted identity / no-login mode. This mode may trust identity headers only when the deployment has an explicitly configured trusted boundary.

The specification for upstream trusted identity must define:

- Which headers are accepted.
- How local development simulates identity.
- Which reverse proxies or access layers are considered trusted.
- How header spoofing is prevented.
- What happens when identity context is absent or malformed.

OIDC, SAML, passkeys, service credentials, and password authentication must be explicit adapters. Password authentication must remain opt-in if it is ever implemented.

## 5. Authorization

Authorization must be deny-by-default. Generated handlers must call policy checks before mutations and before broad data access.

Policies must receive only documented inputs. Policy outputs must include:

- Decision: allow or deny.
- Reason code.
- Optional safe explanation.
- Audit metadata.

Denied requests must not reveal sensitive record existence across tenant boundaries.

## 6. Tenancy

Every resource must use exactly one tenancy mode:

- `global`: shared resource with explicit policy protection.
- `tenant_scoped`: rows belong to exactly one tenant.
- `tenant_partitioned`: data is isolated by schema or database.
- `system`: framework or operational resource.

For `tenant_scoped` resources, generated code must include tenant constraints in reads, lists, updates, deletes, relationship traversal, uniqueness checks, and existing-record lookups.

## 7. SQL safety

Generated code must not interpolate untrusted values into SQL strings. SQL execution must use parameter binding for values and safe identifier handling for generated identifiers.

Custom SQL escape hatches must be:

- Explicitly named.
- Easy to search for.
- Covered by tests.
- Reviewed for tenant constraints.
- Excluded from default generated paths.

## 8. Secrets and redaction

Secrets must not appear in:

- Plans.
- Logs.
- Error messages.
- OpenAPI files.
- JSON Schema files.
- Generated examples.
- Deployment manifests committed to source control.

When a secret value affects review, output should show a redacted placeholder and metadata such as source name or required variable name.

## 9. Audit

Generated applications must write audit events for security-relevant operations. Audit records should include:

- Event ID.
- Timestamp.
- Correlation ID.
- Principal ID.
- Tenant ID when applicable.
- Resource and operation.
- Decision or result.
- Safe reason code.
- Redacted change summary when applicable.

Audit persistence failures during mutations must be specified before implementation. The default should prefer failing closed for security-sensitive mutations.

## 10. Supply-chain security

Generated projects must include:

- Lockfile enforcement.
- Minimal dependencies.
- Dependency review guidance.
- Vulnerability scanning path.
- SBOM generation path.
- Release signing guidance.
- Template integrity expectations.

Dependencies that affect authentication, authorization, SQL generation, cryptography, or deployment must receive heightened review.

## 11. Security acceptance criteria

Phase 1 code must prove that:

- Missing policies deny access.
- Cross-tenant reads and writes fail.
- SQL injection payloads are parameterized or rejected.
- Secrets are redacted from plans and logs.
- Password login is absent from the default template.
- Audit events are emitted for allowed and denied security-relevant operations.
