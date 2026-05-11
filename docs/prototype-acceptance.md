# Daicho Prototype Acceptance Plan

Status: Draft 0.4
Phase: 0

## 1. Purpose

This document defines the acceptance criteria for the first Daicho prototype, including AI-assisted development assumptions, web/API product delivery, protected ingress, and runtime linkage evidence.

## 2. End-to-end scenario

The prototype passes when a developer or AI-agent-oriented command-line demo can:

1. Use the clone-first starter project.
2. Define one tenant-scoped `customer` resource in strict canonical JSON.
3. Define operation policies in strict policy JSON.
4. Validate resources, tenancy, policy bindings, package-manager state, SBOM generation, and vulnerability scan inputs.
5. Export OpenAPI, JSON Schema, TypeScript model types, and generated SQL migration artifacts.
6. Start PostgreSQL and the application locally.
7. Simulate upstream trusted identity with active and disabled account states using a principal shape compatible with future SAML assertions and SCIM provisioning events.
8. Prove disabled principals are rejected before resource policy evaluation.
9. Create, read, list, update, and delete customer records.
10. Prove cross-tenant reads and writes fail.
11. Prove missing or denying policies block access.
12. Prove audit events are written for CRUD, authorization denial, and disabled-principal rejection.
13. Demonstrate that a destructive migration plan cannot be applied without checksum-bound human approval.
14. Generate a CycloneDX SBOM and run vulnerability scanning from the command line.
15. Run all tests from the command line.
16. Serve the resource through a web API without requiring product users to run the Daicho CLI.
17. Produce Docker Compose deployment files and a redacted plan.
18. Document protected-ingress assumptions and warn against direct public origin exposure.
19. Record how the application links to the Daicho runtime, including exact version or commit identity.

## 3. Required checks

The prototype must check:

- Resource JSON validation.
- Policy JSON validation.
- Generated artifact freshness.
- Migration presence and safety classification.
- Tenant-safe query behavior.
- Deny-by-default authorization.
- Disabled-principal rejection before policy evaluation using SAML/SCIM-compatible principal and lifecycle fields.
- Policy tests.
- Structured logs.
- Audit records.
- OpenAPI export.
- JSON Schema export.
- Docker Compose deployment preparation.
- Product-user web API access without Daicho CLI usage.
- Protected-ingress configuration classification: verified, trusted-network-only, or unverified.
- Runtime linkage version or commit reporting.
- Absence of password login from the default starter.
- `pnpm` frozen-lockfile install path.
- CycloneDX SBOM generation and OSV-Scanner vulnerability scan path.

## 4. Security failure cases

The demo must show that:

- Requests without identity fail.
- Disabled, suspended, or deprovisioned principals fail before policy evaluation.
- Tenant-scoped requests without tenant context fail.
- Cross-tenant record access fails.
- Missing policy bindings fail validation or deny access.
- SQL injection-like inputs do not alter query structure.
- Custom SQL outside the approved helper fails validation.
- Destructive migration application fails without matching human approval.
- Secrets do not appear in plans or logs.
- Spoofed upstream identity headers fail outside the configured trusted boundary.
- Unverified ingress fails upstream trusted identity checks.

## 5. Acceptance evidence

A passing prototype must provide:

- Demo commands.
- Relevant source and generated/exported files.
- Migration plan and checksum-bound approval example.
- SBOM and vulnerability scan output.
- Test output.
- Example HTTP requests and responses.
- Example audit records, including a disabled-principal rejection event.
- Example redacted plan output.
- Docker Compose files.
- Protected-ingress notes or diagnostics.
- Runtime linkage evidence.
