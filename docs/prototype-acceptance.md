# Daicho Prototype Acceptance Plan

Status: Draft 0.2
Phase: 0

## 1. Purpose

This document defines the acceptance criteria for the first Daicho prototype.

## 2. End-to-end scenario

The prototype passes when a command-line demo can:

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
12. Produce Docker Compose deployment files and a redacted plan.

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
- Absence of password login from the default starter.

## 4. Security failure cases

The demo must show that:

- Requests without identity fail.
- Tenant-scoped requests without tenant context fail.
- Cross-tenant record access fails.
- Missing policy bindings fail validation or deny access.
- SQL injection-like inputs do not alter query structure.
- Secrets do not appear in plans or logs.

## 5. Acceptance evidence

A passing prototype must provide:

- Demo commands.
- Relevant source and generated/exported files.
- Test output.
- Example HTTP requests and responses.
- Example audit records.
- Example redacted plan output.
- Docker Compose files.
