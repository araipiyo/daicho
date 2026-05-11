# Daicho Architecture Specification

Status: Draft 0.2
Phase: 0

## 1. Purpose

This document defines the Phase 1 architecture. Daicho is a runtime-library-first framework with CLI checks and reviewable source artifacts.

## 2. Architecture goals

Daicho must be:

- Command-first and GUI-free for development.
- Secure by default and explicit about security-sensitive behavior.
- Text-first, deterministic, and review-friendly.
- PostgreSQL-centered for data, migrations, tenancy, and audit.
- Portable across local containers, managed PostgreSQL, Kubernetes, and cloud container platforms.

## 3. Components

```text
Project source  ->  Daicho CLI checks/exports  ->  Daicho runtime library
resources          plans, diagnostics, schemas      HTTP API, policy, audit
policies                                           |
config                                             v
Docker files                                  PostgreSQL
```

## 4. Project inputs

A project must contain explicit source files for:

- Configuration.
- Resources.
- Policies.
- Policy and integration tests.
- Environment templates.
- Deployment templates or target configuration.

Missing required inputs must produce machine-readable diagnostics.

## 5. CLI pipeline

The CLI pipeline is:

1. Discover project root.
2. Load configuration.
3. Load resources and policies.
4. Compile an intermediate representation.
5. Validate security and tenancy invariants.
6. Emit diagnostics, exports, or plans.
7. Apply no infrastructure changes unless an explicit command requires it.

JSON output is required for automation. Plans must not contain secrets.

## 6. Intermediate representation

The intermediate representation must include:

- Resource names and source locations.
- Field metadata.
- Tenancy metadata.
- Database mappings.
- Indexes, constraints, and relationships.
- API exposure.
- Policy bindings.
- Audit settings.

It must be serializable and secret-free.

## 7. Runtime request flow

A request must flow in this order:

1. Parse metadata and correlation ID.
2. Extract principal from configured identity mode.
3. Resolve tenant context.
4. Validate request shape.
5. Evaluate policy.
6. Execute tenant-safe database work, using a transaction for mutations.
7. Persist audit events.
8. Emit structured logs.
9. Return a structured response or error.

## 8. PostgreSQL boundary

PostgreSQL is the system of record. The database layer must provide:

- Parameterized SQL execution.
- Safe generated identifier handling.
- Tenant-aware query construction.
- Source-controlled migrations.
- Transaction boundaries for mutations.
- Audit persistence.

Generated or starter code must not concatenate untrusted values into SQL.

## 9. Policy boundary

Policies must be called through a stable interface with:

- Principal.
- Tenant context.
- Resource name.
- Operation.
- Existing record values when needed.
- Proposed record values for creates and updates.
- Request metadata required by policy.

Policies return allow or deny with safe reason codes for audit.

## 10. Adapter boundary

Adapters are optional modules. The core runtime must not require a hosted Daicho service.

Adapter categories include identity providers, object storage, queues, email, deployment targets, and observability exporters. Each adapter must declare configuration, secrets, capabilities, and failure behavior.

## 11. Observability

Generated applications must provide:

- Structured logs with correlation IDs.
- Secret redaction.
- Health endpoint.
- Readiness endpoint.
- Audit records for security-relevant events.

## 12. Acceptance

The architecture is acceptable when it explains one tenant-scoped CRUD request from HTTP ingress through identity, tenant resolution, policy, SQL, audit, logs, and response serialization without a GUI or hidden service.
