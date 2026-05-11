# Daicho Prototype Acceptance Plan

Status: Draft 0.1  
Phase: 0

## 1. Purpose

This document defines the acceptance criteria for the first Daicho prototype.

## 2. End-to-end scenario

The prototype is acceptable when it can demonstrate this scenario entirely from the command line:

1. Initialize a new Daicho project.
2. Define one tenant-scoped `customer` resource.
3. Validate the resource and policy bindings.
4. Generate TypeScript runtime code, SQL migrations, OpenAPI, and JSON Schema.
5. Start PostgreSQL and the generated application locally.
6. Simulate an upstream trusted identity.
7. Create, read, list, update, and delete customer records.
8. Prove cross-tenant reads and writes fail.
9. Prove missing or denying policies block access.
10. Prove audit events are written.
11. Run all tests from the command line.
12. Produce Docker Compose deployment files.

## 3. Required checks

The prototype must include checks for:

- Project initialization.
- Resource validation.
- Migration generation.
- Tenant-safe query generation.
- Deny-by-default authorization.
- Policy test execution.
- Structured logs.
- Audit records.
- OpenAPI export.
- JSON Schema export.
- Docker Compose generation.
- Absence of password login from the default template.

## 4. Security failure cases

The acceptance demo must show that:

- Requests without identity fail.
- Requests without tenant context fail for tenant-scoped resources.
- Cross-tenant record access fails.
- Missing policy bindings fail validation or deny access.
- SQL injection-like inputs do not alter query structure.
- Secrets do not appear in plans or logs.

## 5. Acceptance evidence

A passing prototype should provide:

- Commands used for the demo.
- Generated files.
- Test output.
- Example HTTP requests and responses.
- Example audit records.
- Example redacted plan output.
- Docker Compose files.
