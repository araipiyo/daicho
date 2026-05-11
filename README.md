# Daicho

Daicho is an open-source framework specification for building AI-era business tools and operational CRUD applications with a command-first developer experience, secure identity by default, and low-operation deployment paths.

The project is intentionally starting with a specification before implementation so that architecture, security, and operability can be reviewed globally and collaboratively.

## Vision

Build a TypeScript-centered framework that lets teams create internal business systems, back-office workflows, data stewardship tools, and operational APIs without relying on development GUIs or mandatory administrative GUIs.

Daicho assumes that modern builders use AI coding agents, terminal workflows, code review, infrastructure-as-code, and policy automation. The framework should therefore be optimized for text, schemas, generated code, repeatable templates, and secure defaults rather than drag-and-drop screens.

## Non-goals

Daicho is not intended to be:

- A no-code or low-code visual app builder.
- A GUI-first admin panel generator.
- A password-login starter kit.
- A single-cloud platform-as-a-service.
- A framework that hides security-critical behavior behind opaque plugins.

## Core principles

1. **Command-first, AI-friendly development**: all project definition, generation, migrations, policies, and deployment workflows must be available through plain text files and CLI commands.
2. **No development GUI**: visual development tools are out of scope. The source of truth must remain code, schemas, migrations, policies, and tests.
3. **No default administrative GUI**: generated management screens are not a default feature. Operators should use APIs, CLIs, auditable workflows, and optional separately maintained UIs only when truly required.
4. **Secure identity by default**: SSO, SAML, OIDC, and passkeys must be first-class. Password login must be disabled by default.
5. **PostgreSQL as the system of record**: PostgreSQL is the primary database target for transactional data, migrations, row-level policies, and audit records.
6. **Multi-cloud and portable**: deployments should work across major clouds and self-hosted environments using open standards and minimal provider lock-in.
7. **Minimal dependencies**: prefer the platform, small internal modules, and carefully selected dependencies. Every dependency must justify its operational and supply-chain risk.
8. **Multi-tenant capable**: the architecture must support single-tenant, shared-database multi-tenant, and stronger isolation models.
9. **Auditable operations**: business operations, administrative changes, access decisions, and generated artifacts should be traceable.
10. **Progressive adoption**: teams should be able to adopt Daicho for one service, one workflow, or one tenant model without migrating their whole stack.

## Target use cases

- Internal business operations APIs.
- CRUD-oriented workflow services.
- Approval, review, and exception-handling systems.
- Data quality and stewardship tools.
- Multi-tenant SaaS back-office services.
- AI-agent-operated operational systems where humans review plans, diffs, and logs instead of clicking through dashboards.

## High-level architecture

```text
+----------------------+      +-----------------------+
| AI / Human CLI       | ---> | Daicho CLI            |
| code review, shell   |      | generate, migrate,    |
| CI/CD, IaC           |      | test, deploy          |
+----------------------+      +-----------+-----------+
                                      |
                                      v
+----------------------+      +-----------------------+
| Identity Providers   | ---> | Application Runtime   |
| OIDC, SAML, Passkey  |      | TypeScript services,  |
| enterprise SSO       |      | APIs, jobs, policies  |
+----------------------+      +-----------+-----------+
                                      |
                                      v
+----------------------+      +-----------------------+
| Object Storage /     | <--> | PostgreSQL            |
| Queues / Email       |      | data, migrations,     |
| optional adapters    |      | tenancy, audit        |
+----------------------+      +-----------------------+
```

## Proposed components

### 1. Daicho CLI

The CLI is the primary interface for developers, operators, and AI agents.

Required capabilities:

- Initialize projects from templates.
- Generate modules from declarative resource definitions.
- Create and run PostgreSQL migrations.
- Validate schemas, policies, dependency constraints, and deployment manifests.
- Produce machine-readable plans before applying changes.
- Export OpenAPI and JSON Schema artifacts.
- Run local tests without requiring a GUI.
- Prepare deployment bundles for supported targets.

### 2. Resource definition format

Daicho resources should be defined as text-first schemas. The initial format should be TypeScript modules with a strict subset that can be statically analyzed.

A resource should describe:

- Database table mapping.
- Fields, constraints, indexes, and relationships.
- Validation rules.
- Tenant scope.
- Authorization policies.
- Audit behavior.
- API exposure rules.
- Workflow hooks.

Example direction:

```ts
export const customer = resource({
  name: "customer",
  tenancy: "tenant_scoped",
  fields: {
    id: uuid().primaryKey(),
    tenantId: uuid().tenantKey(),
    name: text().min(1).max(200),
    email: email().optional(),
    createdAt: timestamp().defaultNow(),
  },
  access: {
    read: allow("tenant.member"),
    create: allow("tenant.operator"),
    update: allow("tenant.operator"),
    delete: deny(),
  },
  audit: {
    create: true,
    update: true,
    delete: true,
  },
});
```

### 3. Runtime

The runtime should be a small TypeScript server package that exposes generated APIs and executes policies.

Preferred characteristics:

- Runs on standard Node.js LTS runtimes.
- Avoids framework lock-in where possible.
- Supports HTTP APIs as the default interface.
- Can be packaged as containers or serverless-compatible handlers.
- Keeps business logic explicit and testable.
- Emits structured logs and traces.

### 4. Authentication and identity

Authentication must be secure by default and enterprise-ready.

Required support:

- OIDC / OAuth 2.0 provider integration.
- SAML 2.0 provider integration for enterprise SSO.
- Passkey / WebAuthn support.
- SCIM-compatible identity provisioning as a future capability.
- Session management with secure cookies where browser clients are used.
- Service-to-service authentication for automation and agents.

Default posture:

- Password login is disabled by default.
- Local password authentication, if implemented, must be an explicit opt-in module.
- MFA requirements should be delegated to identity providers when possible.
- Identity claims must be normalized before policy evaluation.

### 5. Authorization

Authorization should be policy-driven and testable.

The first implementation should support:

- Role-based access control for simple cases.
- Attribute-based checks for tenant, ownership, and workflow state.
- Resource-level CRUD policies.
- Field-level read/write restrictions where feasible.
- Deny-by-default behavior.
- Policy test fixtures.

Authorization must not depend on generated UI assumptions.

### 6. Multi-tenancy

Daicho must support multiple tenancy models because different organizations have different isolation requirements.

Initial models:

1. **Single tenant**: one deployment and one database for one organization.
2. **Shared database, shared schema**: rows are scoped by `tenant_id`; suitable for many SaaS products.
3. **Shared database, separate schemas**: stronger namespace isolation while sharing infrastructure.
4. **Separate database per tenant**: stronger isolation for regulated or high-value tenants.

Baseline requirements:

- Every tenant-scoped resource must declare its tenant key.
- All generated queries must include tenant constraints where applicable.
- Migration tooling must support tenant-aware rollout plans.
- Audit logs must include tenant identifiers.
- Tenant context must be derived from trusted identity/session state, not user input alone.

### 7. PostgreSQL data layer

PostgreSQL is the required transactional database.

Required capabilities:

- Declarative migration generation.
- Manual migration escape hatch.
- Transactional CRUD operations.
- Optimistic locking support.
- JSONB support for carefully bounded extension data.
- Advisory locks for migration and job coordination where appropriate.
- Optional PostgreSQL row-level security integration.
- Audit log tables generated by default for sensitive resources.

### 8. Deployment templates

Daicho should include simple, auditable deployment templates rather than a hosted proprietary control plane.

Initial targets:

- Docker Compose for local and small self-hosted deployments.
- Kubernetes manifests or Helm chart for portable production deployments.
- AWS template using managed PostgreSQL and container/serverless runtime options.
- Google Cloud template using managed PostgreSQL-compatible services and containers.
- Azure template using managed PostgreSQL and containers.
- Fly.io / Render-style lightweight templates may be considered for smaller teams.

Deployment guidance:

- Prefer managed PostgreSQL when possible.
- Prefer managed container platforms over hand-managed virtual machines.
- Keep secrets in provider secret managers or sealed secret workflows.
- Do not require a Daicho-hosted control plane.

### 9. Supply-chain security

Daicho must treat dependency and build integrity as core product features.

Requirements:

- Lockfiles are mandatory for generated projects.
- Dependency updates must be explicit and reviewable.
- Minimize runtime dependencies.
- Prefer direct dependencies over deep plugin chains.
- Publish software bills of materials for releases.
- Sign release artifacts where feasible.
- Support provenance metadata in CI.
- Provide a dependency allowlist / denylist mechanism.
- Avoid executing remote code during project generation.
- Make generated code deterministic where possible.

### 10. Observability and audit

Daicho should make operations understandable without a mandatory GUI.

Required outputs:

- Structured JSON logs.
- Request IDs and tenant IDs in logs.
- Audit events for authentication, authorization failures, administrative operations, and data changes.
- Health and readiness endpoints.
- Metrics endpoint compatible with common collectors.
- OpenTelemetry-compatible tracing as an optional integration.

### 11. Optional user interfaces

Daicho does not generate a management GUI by default. However, some organizations may need human-facing review or data-entry tools.

If UI support is added later, it should be:

- Optional.
- Generated from the same schemas and policies.
- Packaged separately from the core runtime.
- Disabled by default.
- Audited and policy-aware.
- Replaceable by custom front ends.

## TypeScript and dependency strategy

Implementation should prioritize:

- TypeScript for framework code and generated applications.
- Node.js LTS compatibility.
- Standard Web APIs where available.
- Small, well-maintained libraries only when platform APIs are insufficient.
- Clear module boundaries so cloud adapters are optional.
- No mandatory ORM if direct SQL generation can remain safe and maintainable.
- Strict TypeScript settings.
- Reproducible package management.

## Project structure proposal

```text
daicho/
  packages/
    cli/              # command-line interface
    core/             # schema, policy, and generation primitives
    runtime/          # HTTP runtime and execution engine
    postgres/         # PostgreSQL adapter and migration tools
    auth/             # identity normalization and auth adapters
    deploy/           # deployment template generation
  templates/
    minimal-api/
    multi-tenant-saas/
    enterprise-sso/
  examples/
    customer-support/
    approvals/
  docs/
    specification.md
    security.md
    tenancy.md
    deployment.md
```

## Initial roadmap

### Phase 0: Specification and design review

- Publish this specification.
- Define threat model.
- Define resource schema format.
- Define policy model.
- Define migration strategy.
- Define supported deployment targets.

### Phase 1: Minimal CLI and runtime prototype

- Project initializer.
- One resource definition format.
- PostgreSQL migration generation.
- Basic HTTP CRUD API generation.
- Tenant-scoped query enforcement.
- Structured logging.

### Phase 2: Secure identity and policy engine

- OIDC integration.
- SAML integration design and adapter boundary.
- Passkey integration design.
- Deny-by-default authorization.
- Policy test runner.
- Audit event model.

### Phase 3: Deployment templates

- Docker Compose template.
- Kubernetes template.
- One managed-cloud reference template.
- Secrets handling documentation.
- Backup and restore documentation.

### Phase 4: Hardening

- Supply-chain security checks.
- Release signing and provenance.
- SBOM publishing.
- Performance benchmarks.
- Multi-tenant migration tooling.
- Operational runbooks.

## Open design questions

- Should the first schema format be TypeScript-only, YAML/JSON, or both?
- Should Daicho generate SQL directly or use a small query builder internally?
- How much PostgreSQL row-level security should be generated by default?
- Which identity adapter should be implemented first: OIDC, SAML, or passkeys?
- What is the minimum viable deployment target for production?
- Should optional UI generation live in this repository or a separate project?

## Contributing

This repository is currently in the design stage. Contributions should focus on improving the specification, identifying security risks, validating deployment assumptions, and proposing minimal implementation paths.

Please keep proposals text-first, auditable, and compatible with AI-agent-assisted development workflows.
