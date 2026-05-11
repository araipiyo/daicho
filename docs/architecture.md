# Daicho Architecture Specification

Status: Draft 0.3
Phase: 0

## 1. Purpose

This document defines the Phase 1 architecture. Daicho is a runtime-library-first framework with AI-assisted development workflows, CLI checks, reviewable source artifacts, and user-facing web application or web API deliverables.

## 2. Architecture goals

Daicho must be:

- AI-agent-first and command-capable for development, with humans reviewing source, plans, tests, and diffs.
- GUI-free for development and free of any default administrative GUI.
- Secure by default and explicit about security-sensitive behavior.
- Text-first, deterministic, and review-friendly.
- PostgreSQL-centered for data, migrations, tenancy, and audit.
- Portable across local containers, managed PostgreSQL, Kubernetes, and cloud container platforms.
- Clear that the Daicho CLI is for developers, AI agents, CI, and limited operator tasks, not product users.
- Designed to run behind protected ingress rather than exposing an origin server directly.

## 3. Components

```text
Project source  ->  AI agent / CI / Daicho CLI  ->  Daicho runtime library
resources          checks, plans, diagnostics       web app/API, policy, audit
policies             exports, deployment prep       |
config                                             v
Docker files        protected ingress          PostgreSQL
```

## 4. Development and product interfaces

Developers are expected to use AI coding agents such as Codex or Claude Code, source control, code review, CI, and the Daicho CLI. Humans may run CLI commands directly for local checks, diagnostics, exports, and deployment preparation, but routine development should be safe for AI-agent execution and human review.

Product users interact with Daicho-built applications through web applications, web APIs, background integrations, or service-to-service interfaces. The runtime architecture must not require product users to install the Daicho CLI, inspect plans, or modify source files.

## 5. Project inputs

A project must contain explicit source files for:

- Configuration.
- Resources.
- Policies.
- Policy and integration tests.
- Environment templates.
- Deployment templates or target configuration.

Missing required inputs must produce machine-readable diagnostics.

## 6. CLI pipeline

The CLI pipeline is:

1. Discover project root.
2. Load configuration.
3. Load resources and policies.
4. Compile an intermediate representation.
5. Validate security and tenancy invariants.
6. Emit diagnostics, exports, or plans.
7. Apply no infrastructure changes unless an explicit command requires it.

JSON output is required for automation. Plans must not contain secrets.

## 7. Intermediate representation

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

## 8. Runtime request flow

A request must flow in this order:

1. Parse metadata and correlation ID.
2. Validate the configured ingress trust boundary when the identity mode depends on upstream trusted identity.
3. Extract principal from configured identity mode.
4. Resolve tenant context.
5. Validate request shape.
6. Evaluate policy.
7. Execute tenant-safe database work, using a transaction for mutations.
8. Persist audit events.
9. Emit structured logs.
10. Return a structured response or error.

## 9. PostgreSQL boundary

PostgreSQL is the system of record. The database layer must provide:

- Parameterized SQL execution.
- Safe generated identifier handling.
- Tenant-aware query construction.
- Source-controlled migrations.
- Transaction boundaries for mutations.
- Audit persistence.

Generated or starter code must not concatenate untrusted values into SQL.

## 10. Policy boundary

Policies must be called through a stable interface with:

- Principal.
- Tenant context.
- Resource name.
- Operation.
- Existing record values when needed.
- Proposed record values for creates and updates.
- Request metadata required by policy.

Policies return allow or deny with safe reason codes for audit.

## 11. Adapter boundary

Adapters are optional modules. The core runtime must not require a hosted Daicho service.

Adapter categories include identity providers, protected ingress providers, object storage, queues, email, deployment targets, and observability exporters. Each adapter must declare configuration, secrets, capabilities, trust assumptions, and failure behavior.

## 12. Protected ingress boundary

Production deployments should place Daicho applications behind an explicit access layer such as Cloudflare Access, Tailscale, an identity-aware proxy, API gateway, private service mesh, or equivalent zero-trust product. Direct public exposure of the origin server is discouraged even for public applications; public traffic should normally terminate at an edge, gateway, or proxy that enforces rate limits, TLS, identity, bot controls, WAF policy, or equivalent controls before forwarding to the origin.

The runtime can only confirm an access path when the access layer exposes verifiable evidence. Supported patterns should include signed access-layer JWTs, mTLS client certificates, signed service tokens, or provider-specific proofs validated against a configured issuer. Network-only trust may be accepted only when deployment configuration restricts source networks or proxy identities and Daicho checks can validate the declared assumptions. Unsigned identity headers from arbitrary clients must never be trusted.

## 13. Runtime linkage boundary

Phase 1 may keep the Daicho runtime in the clone-first starter repository, but application code must import it through stable runtime APIs and must not duplicate security-critical internals. The repository layout must make framework-owned runtime files distinguishable from application-owned files.

The preferred future boundary is a separately versioned runtime dependency with immutable version pins, provenance, integrity verification, and a documented security update workflow. Candidate linkage mechanisms include a package registry artifact, signed release archive, Git submodule, or another auditable mechanism. The architecture must preserve the ability to patch the runtime across applications without relying on unreviewed local forks.

## 14. Observability

Generated applications must provide:

- Structured logs with correlation IDs.
- Secret redaction.
- Health endpoint.
- Readiness endpoint.
- Audit records for security-relevant events.
- Ingress trust diagnostics that report whether runtime identity is verified, trusted-network-only, or unverified.

## 15. Acceptance

The architecture is acceptable when it explains one tenant-scoped CRUD request from HTTP ingress through identity, tenant resolution, policy, SQL, audit, logs, and response serialization without a GUI, hidden service, product-user CLI step, or directly exposed origin dependency.
