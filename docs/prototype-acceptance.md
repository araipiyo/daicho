# Daicho Prototype Acceptance Plan

Status: Draft 0.3
Phase: 0

## 1. Purpose

This document defines the acceptance criteria for the first Daicho prototype, including AI-assisted development assumptions, web/API product delivery, protected ingress, and runtime linkage evidence.

## 2. End-to-end scenario

The prototype passes when a developer or AI-agent-oriented command-line demo can:

1. Use the clone-first starter project.
2. Define one tenant-scoped `customer` resource.
3. Validate resources, tenancy, and policy bindings.
4. Export OpenAPI and JSON Schema.
5. Start PostgreSQL and the application locally.
6. Simulate upstream trusted identity.
7. Create, read, list, update, and delete customer records.
8. Prove cross-tenant reads and writes fail.
9. Prove missing or denying policies block access.
10. Prove audit events are written.
11. Run all tests from the command line.
12. Serve the resource through a web API without requiring product users to run the Daicho CLI.
13. Produce Docker Compose deployment files and a redacted plan.
14. Document protected-ingress assumptions and warn against direct public origin exposure.
15. Record how the application links to the Daicho runtime, including exact version or commit identity.

## 3. Required checks

The prototype must check:

- Resource validation.
- Migration presence and safety classification.
- Tenant-safe query behavior.
- Deny-by-default authorization.
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

## 4. Security failure cases

The demo must show that:

- Requests without identity fail.
- Tenant-scoped requests without tenant context fail.
- Cross-tenant record access fails.
- Missing policy bindings fail validation or deny access.
- SQL injection-like inputs do not alter query structure.
- Secrets do not appear in plans or logs.
- Spoofed upstream identity headers fail outside the configured trusted boundary.
- Unverified ingress fails upstream trusted identity checks.

## 5. Acceptance evidence

A passing prototype must provide:

- Demo commands.
- Relevant source and generated/exported files.
- Test output.
- Example HTTP requests and responses.
- Example audit records.
- Example redacted plan output.
- Docker Compose files.
- Protected-ingress notes or diagnostics.
- Runtime linkage evidence.
