# Daicho Framework Specification

Status: Draft 0.1

> AI makes code cheap. Daicho makes operational code trustworthy.

## 1. Purpose

Daicho is an open-source framework for AI-era business tools and CRUD-oriented operational systems. Its purpose is to provide a security-first, text-first, TypeScript-centered foundation for building APIs and workflows that are usually served by internal admin panels, while avoiding GUI-first development and avoiding default administrative GUIs. Daicho treats operational code as production-critical infrastructure: generated code must be reviewable, policy-aware, tenant-safe, and secure by default.

## 2. Design constraints

Daicho implementations must satisfy these constraints:

- Security first: every generator, runtime path, adapter, and template must prefer explicit, reviewable, least-privilege behavior over convenience.
- Secure by default: default projects must avoid password login, unsafe SQL construction, cross-tenant access, hidden infrastructure, and unnecessary administrative surfaces.
- Development must be possible entirely through source files, CLI commands, tests, and CI/CD.
- A development GUI must not be required or provided as a core workflow.
- Administrative GUI generation must not be enabled by default.
- Password login must be disabled by default, and the first authentication mode may assume an upstream trusted boundary such as Cloudflare Access or another external access proxy.
- PostgreSQL must be the primary transactional database.
- Multi-tenancy must be designed into the resource, policy, audit, and migration layers.
- Deployment must be possible without a proprietary hosted control plane.
- Dependencies must be minimized, pinned, reviewed, and replaceable where possible.
- Security-sensitive behavior must remain understandable in generated source, tests, plans, and migrations.

## 3. Terminology

- **Resource**: a declarative description of a business entity, its persistence, validation, API exposure, authorization, and audit behavior.
- **Tenant**: an isolation boundary for customers, organizations, departments, or environments.
- **Policy**: a declarative or code-defined authorization rule evaluated before an operation.
- **Adapter**: an optional integration module for a cloud provider, identity provider, queue, storage system, or deployment target.
- **Generated artifact**: source code, SQL, OpenAPI documents, JSON Schemas, manifests, or plans produced by the CLI.
- **Plan**: a machine-readable and human-reviewable description of proposed changes before execution.

## 4. Functional requirements

### 4.1 CLI

The CLI must provide commands for:

- Project initialization.
- Resource validation.
- Code generation.
- Migration generation and execution.
- Policy testing.
- OpenAPI and JSON Schema export.
- Deployment template generation.
- Dependency and supply-chain checks.

Commands that mutate infrastructure or databases should support a dry-run mode and should emit a plan.

### 4.2 Resource model

Resources must define:

- Stable resource name.
- Field definitions and validation constraints.
- Database table mapping.
- Tenant scope.
- Index requirements.
- Relationships.
- CRUD operations exposed through APIs.
- Authorization policy bindings.
- Audit requirements.

Resource definitions should be statically analyzable and deterministic. The initial implementation may support TypeScript schemas, YAML/JSON schemas, or both, provided the chosen format remains text-first, deterministic, and reviewable.

### 4.3 API runtime

The runtime must support:

- HTTP APIs for CRUD operations.
- Request validation.
- Response serialization.
- Tenant context propagation.
- Policy evaluation before data access.
- Transaction handling.
- Structured errors.
- Structured logs.
- Health and readiness endpoints.

GraphQL, event APIs, or UI-specific endpoints may be added later, but they are not required for the first implementation.

### 4.4 Authentication

Daicho must support identity integrations through explicit adapters. The first implementation should support a no-login application mode that trusts a validated upstream identity or access boundary, such as Cloudflare Access or an equivalent reverse-proxy identity layer.

Required adapter categories:

- OIDC / OAuth 2.0.
- SAML 2.0.
- Passkey / WebAuthn.
- Service credentials for automation.
- Upstream trusted identity / no-login mode for deployments protected by an external access layer.

Password authentication must be implemented only as an opt-in module, if it is implemented at all. Built-in OIDC, SAML, and passkey adapters may follow after the no-login/upstream-identity mode.

### 4.5 Authorization

Authorization must be deny-by-default. Policies must be testable without a running browser or GUI.

Minimum policy inputs:

- Principal identity.
- Tenant context.
- Resource name.
- Operation.
- Existing record values when available.
- Proposed record values for create and update operations.
- Request metadata required by policy.

### 4.6 Tenancy

Every resource must declare one of these tenancy modes:

- `global`: shared across all tenants.
- `tenant_scoped`: rows or objects belong to exactly one tenant.
- `tenant_partitioned`: data is isolated by schema or database.
- `system`: internal framework or operational resource.

Generated code must prevent accidental cross-tenant access by construction.

### 4.7 PostgreSQL

The PostgreSQL layer must provide:

- Safe SQL generation through a minimal internal query builder or equivalent guardrail that at least prevents SQL injection and enforces security-critical query invariants.
- Migration files committed to source control.
- Transaction boundaries for mutations.
- Tenant-aware query constraints.
- Optional row-level security generation as a future capability, not a first-prototype default.
- Audit event persistence.
- Backup and restore documentation.

### 4.8 Deployment

Deployment templates must avoid hidden infrastructure. Generated templates should be readable and editable.

Required template qualities:

- No required Daicho-hosted service.
- Clear secret inputs.
- Managed PostgreSQL support.
- Container-based runtime support, with Docker Compose as the minimum local deployment target for early testing.
- Health checks.
- Rollback guidance.
- Environment-specific configuration files.

### 4.9 Supply-chain security

Generated projects must include:

- Lockfile enforcement.
- Dependency review guidance.
- Minimal default dependency set.
- Reproducible build recommendations.
- SBOM generation path.
- Release signing path.
- CI checks for known vulnerabilities.

## 5. Non-functional requirements

- **Portability**: applications should run locally, in containers, on Kubernetes, and on major cloud container platforms.
- **Operability**: common operations should be scriptable and observable without dashboards.
- **Security first**: security requirements should shape schema design, generation, query construction, authentication boundaries, policy evaluation, and deployment templates before convenience features.
- **Secure by default**: default projects should reduce exposure for small teams and remain extensible for enterprises without requiring hidden administrative services.
- **Performance**: generated CRUD paths should avoid unnecessary abstraction layers.
- **Maintainability**: generated code should be understandable and safe to review.
- **International adoption**: documentation, examples, errors, and generated APIs should be English-first initially and localization-ready later.

## 6. Threat model summary

Daicho must consider at least these threats:

- Compromised npm dependencies.
- Malicious project templates.
- Cross-tenant data leakage.
- Misconfigured identity providers.
- Authorization bypass through generated routes.
- SQL injection in generated or custom queries.
- Secret leakage in logs, plans, or deployment manifests.
- Replay or misuse of service credentials.
- Unsafe migrations that corrupt tenant data.
- AI-agent mistakes that apply changes without review.

Mitigations should be documented and tested as implementation begins.

## 7. Compatibility targets

Initial compatibility targets should be:

- Node.js LTS.
- TypeScript with strict settings.
- PostgreSQL supported versions that are still maintained by the PostgreSQL project.
- Linux containers.
- Major cloud managed PostgreSQL offerings.

Exact version ranges should be decided when implementation starts.

## 8. Acceptance criteria for the first prototype

The first prototype is acceptable when it can:

1. Initialize a new project from the CLI.
2. Define at least one tenant-scoped resource.
3. Generate PostgreSQL migration SQL.
4. Run a local HTTP CRUD API.
5. Enforce tenant constraints in generated queries.
6. Evaluate a deny-by-default policy.
7. Emit structured logs and audit records.
8. Run tests from the command line.
9. Produce a Docker Compose deployment for local operation.
10. Avoid password login in the default template.
11. Operate behind an upstream trusted identity or access boundary without requiring an in-app login screen.
12. Generate database access through the minimal safe query builder or equivalent guardrail.

## 9. Resolved design decisions

- **First schema format**: TypeScript, YAML/JSON, or both are acceptable. The chosen implementation must remain text-first, deterministic, statically analyzable, and reviewable.
- **SQL generation strategy**: Daicho should include at least a minimal internal query builder or equivalent guardrail for security-critical SQL construction. Security and tenant invariants matter more than avoiding all abstraction.
- **PostgreSQL row-level security**: RLS generation is not required for the first prototype. It should remain a future optional capability.
- **First identity mode**: implement a no-login / upstream trusted identity mode first, assuming deployments can be protected by Cloudflare Access or an equivalent external access layer. OIDC, SAML, and passkeys can follow as explicit adapters.
- **Minimum viable deployment target**: provide Docker Compose or an equivalent local container setup first so teams can test Daicho easily.
- **Optional UI generation**: Daicho itself may not need a management UI. Generated applications should be manageable through APIs, CLIs, and application-specific workflows; any optional UI generator should be separate from the core runtime unless a strong reason emerges.
