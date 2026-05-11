# Daicho Architecture Specification

Status: Draft 0.1  
Phase: 0

## 1. Purpose

This document defines the target architecture for the first Daicho prototype. It is normative for Phase 1 unless superseded by an accepted decision record.

## 2. Architectural goals

Daicho must be:

- Command-first and usable without a development GUI.
- Secure by default and explicit about security-sensitive behavior.
- Text-first, deterministic, and friendly to code review and AI-assisted development.
- PostgreSQL-centered for transactional data, migrations, tenancy, and audit records.
- Portable across local containers, managed PostgreSQL, Kubernetes, and major cloud container platforms.

## 3. Component overview

```text
+-------------------+      +-------------------+      +-------------------+
| Project files     | ---> | Daicho CLI        | ---> | Generated app     |
| resources, policy |      | validate, plan,   |      | HTTP API, jobs,   |
| config, tests     |      | generate, migrate |      | policies, audit   |
+-------------------+      +---------+---------+      +---------+---------+
                                      |                          |
                                      v                          v
                            +-------------------+      +-------------------+
                            | Generated output  |      | PostgreSQL        |
                            | TS, SQL, OpenAPI, |      | data, migrations, |
                            | JSON Schema, IaC  |      | audit             |
                            +-------------------+      +-------------------+
```

## 4. Project inputs

A Daicho project must contain explicit source files for:

- Project configuration.
- Resource definitions.
- Authorization policies.
- Policy tests.
- Environment configuration templates.
- Deployment templates or deployment target configuration.

The CLI must reject missing required inputs with machine-readable diagnostics.

## 5. CLI architecture

The CLI must be implemented as a deterministic pipeline:

1. Discover project root.
2. Load configuration.
3. Load resource and policy sources.
4. Compile resources into an intermediate representation.
5. Validate security and tenancy invariants.
6. Produce a plan for changes.
7. Generate files or apply approved actions only when requested.

The CLI must support JSON output for automation. Mutating commands must support dry-run or plan-only behavior.

## 6. Intermediate representation

The intermediate representation must include:

- Resource names and source locations.
- Field metadata.
- Tenancy mode and tenant key metadata.
- Database table and column mappings.
- Indexes and constraints.
- Relationship definitions.
- API exposure rules.
- Policy bindings.
- Audit settings.

The representation must be serializable for tests and debugging. It must not contain secrets.

## 7. Code generation architecture

Generated code must be reviewable application code, not opaque hidden behavior. Generated modules should be separated into stable areas:

- Resource metadata.
- Validation schemas.
- HTTP handlers.
- Policy invocation wrappers.
- Query builders or repository functions.
- Migration SQL.
- Audit event writers.
- OpenAPI and JSON Schema outputs.

Generated files must identify that they are generated and must include source input hashes when practical.

## 8. Runtime request flow

The generated HTTP runtime must process requests in this order:

1. Parse request metadata and correlation ID.
2. Extract principal from the configured identity mode.
3. Resolve tenant context.
4. Validate request shape.
5. Evaluate authorization policy before mutation or broad data access.
6. Execute tenant-safe database query in a transaction when mutating data.
7. Persist audit events for security-relevant operations.
8. Emit structured logs.
9. Return structured response or structured error.

## 9. PostgreSQL architecture

PostgreSQL is the system of record. The PostgreSQL layer must provide:

- Parameterized SQL execution.
- Tenant-aware query construction.
- Migration files committed to source control.
- Transaction boundaries for mutations.
- Audit event persistence.
- Backup and restore documentation.

Generated code must not concatenate untrusted values into SQL strings.

## 10. Policy engine boundary

Policies must be invoked through a stable interface that receives:

- Principal.
- Tenant context.
- Resource name.
- Operation name.
- Existing record values when needed.
- Proposed record values for create and update operations.
- Request metadata required by policy.

Policies must return explicit allow or deny results with optional reason codes suitable for audit records.

## 11. Adapter architecture

Adapters must be optional modules. The core runtime must not require a proprietary hosted Daicho service. Adapter categories include:

- Identity providers.
- Object storage.
- Queues.
- Email providers.
- Deployment targets.
- Observability exporters.

Adapters must declare configuration, required secrets, capabilities, and failure behavior.

## 12. Observability architecture

Generated applications must emit structured logs with correlation IDs. Logs must not include secrets. Health and readiness endpoints must be available for deployment platforms.

## 13. Architecture acceptance criteria

The architecture is acceptable for Phase 1 when it can explain how a tenant-scoped CRUD request flows from HTTP ingress through identity, tenant resolution, policy, SQL, audit, and response serialization without relying on a GUI or hidden service.
