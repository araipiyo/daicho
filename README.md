# Daicho

> AI makes code cheap. Daicho makes operational code trustworthy.
>
> Vibe-code enterprise apps in a day.
>
> Daicho gives AI agents a secure enterprise application shell:
> SAML, SCIM, RBAC, audit logs, PostgreSQL, and declarative CRUD.

Daicho is an open-source framework specification for building AI-era business tools and operational CRUD applications with an AI-agent-friendly, command-capable developer experience, security-first architecture, secure-by-default behavior, user-facing web application or web API delivery, and low-operation deployment paths.

The project is intentionally starting with a specification before implementation so that architecture, security, secure defaults, and operability can be reviewed globally and collaboratively.

Phase 0 is the specification and implementation-readiness phase. Its detailed scope, deliverables, review workflow, and Phase 1 entry criteria are documented in [`docs/phase-0-specification.md`](docs/phase-0-specification.md).

## Documentation

- [Framework specification](docs/specification.md)
- [Phase 0 specification package](docs/phase-0-specification.md)
- [Architecture specification](docs/architecture.md)
- [Security specification](docs/security.md)
- [Threat model](docs/threat-model.md)
- [Resource model specification](docs/resource-model.md)
- [CLI specification](docs/cli.md)
- [Prototype acceptance plan](docs/prototype-acceptance.md)
- [Decision log](docs/decisions.md)

## Vision

Build a TypeScript-centered framework that lets teams create internal business systems, back-office workflows, data stewardship tools, and operational APIs without relying on development GUIs or mandatory administrative GUIs.

Daicho assumes that modern builders use AI coding agents such as Codex or Claude Code, terminal workflows, code review, infrastructure-as-code, and policy automation. The framework should therefore be optimized for text, schemas, generated code, repeatable templates, security-first decisions, and secure defaults rather than drag-and-drop screens.

## Non-goals

Daicho is not intended to be:

- A no-code or low-code visual app builder.
- A GUI-first admin panel generator.
- A password-login starter kit.
- A single-cloud platform-as-a-service.
- A framework that hides security-critical behavior behind opaque plugins.

## Core principles

1. **AI-agent-first, command-capable development**: all project definition, generation, migrations, policies, and deployment workflows must be available through plain text files and CLI commands suitable for AI agents, CI, and human review.
2. **No development GUI**: visual development tools are out of scope. The source of truth must remain code, schemas, migrations, policies, and tests.
3. **No default administrative GUI**: generated management screens are not a default feature. Operators should use APIs, CLIs, auditable workflows, and optional separately maintained UIs only when truly required; product users should not need the Daicho CLI.
4. **Security first, secure by default**: generators, runtime paths, adapters, and templates must prefer explicit, reviewable, least-privilege behavior. Password login must be disabled by default, unsafe SQL construction must be prevented, and cross-tenant access must be blocked by construction. SSO, SAML, OIDC, passkeys, and upstream trusted identity modes must be first-class over time.
5. **PostgreSQL as the system of record**: PostgreSQL is the primary database target for transactional data, generated or reviewed SQL migrations, tenancy checks, and audit records.
6. **Multi-cloud and portable**: deployments should work across major clouds and self-hosted environments using open standards and minimal provider lock-in.
7. **Protected ingress by default**: deliver web applications and APIs behind Cloudflare Access, Tailscale, identity-aware proxies, API gateways, private service meshes, or equivalent controls rather than exposing origin servers directly.
8. **Minimal dependencies**: prefer the platform, small internal modules, and carefully selected dependencies. Every dependency must justify its operational and supply-chain risk.
9. **Multi-tenant capable**: the architecture must support single-tenant, shared-database multi-tenant, and stronger isolation models.
10. **Auditable operations**: business operations, administrative changes, access decisions, and generated artifacts should be traceable.
11. **Progressive adoption**: teams should be able to adopt Daicho for one service, one workflow, or one tenant model without migrating their whole stack.

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
| AI Agent / CI / CLI  | ---> | Daicho CLI            |
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

The CLI is the primary interface for developers, AI agents, CI, and limited operator workflows. Product users consume Daicho deliverables through web applications, web APIs, service integrations, or background workflows, not through the Daicho CLI.

Required capabilities:

- Validate clone-first starter projects.
- Generate modules from declarative resource definitions.
- Create and run PostgreSQL migrations.
- Validate schemas, policies, dependency constraints, and deployment manifests.
- Produce machine-readable plans before applying changes.
- Export OpenAPI and JSON Schema artifacts.
- Run local tests without requiring a GUI.
- Prepare deployment bundles for supported targets.
- Report protected-ingress assumptions and runtime linkage identity.

### 2. Resource definition format

Daicho resources should be defined as text-first schemas. The initial format may be TypeScript modules, YAML/JSON, or both, as long as it can be statically analyzed and remains deterministic, reviewable, and friendly to AI-assisted workflows.

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

- Upstream trusted identity / no-login mode for deployments protected by Cloudflare Access or an equivalent external access layer.
- OIDC / OAuth 2.0 provider integration.
- SAML 2.0 provider integration for enterprise SSO.
- Passkey / WebAuthn support.
- SCIM-compatible identity provisioning as a future capability.
- Session management with secure cookies where browser clients are used.
- Service-to-service authentication for automation and agents.

Default posture:

- Password login is disabled by default.
- The first implementation should work without an in-app login screen when a trusted external access layer provides identity and access control.
- Local password authentication, if implemented, must be an explicit opt-in module.
- MFA requirements should be delegated to identity providers when possible.
- Identity claims must be normalized before policy evaluation.
- Built-in OIDC, SAML, and passkey adapters may follow after the upstream trusted identity mode.

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
- Optional PostgreSQL row-level security integration as a future capability rather than a first-prototype default.
- Audit log tables generated by default for sensitive resources.
- A minimal internal query builder or equivalent guardrail for security-critical SQL construction, preventing SQL injection and enforcing tenant constraints.

### 8. Deployment templates

Daicho should include simple, auditable deployment templates rather than a hosted proprietary control plane.

Initial targets:

- Docker Compose for local testing and small self-hosted deployments; this is the minimum viable deployment target for the first prototype.
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

Daicho does not generate a management GUI by default, and Daicho itself may not need a management UI. Operators should be able to administer generated applications through their APIs, CLIs, auditable workflows, or application-specific tools. Some organizations may still need human-facing review or data-entry tools.

If UI support is added later, it should be:

- Optional.
- Generated from the same schemas and policies.
- Packaged separately from the core runtime and preferably developed as a separate project unless a strong reason emerges to keep it in this repository.
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
- No mandatory ORM; use a minimal internal query builder or equivalent guardrail where needed to keep SQL generation safe, maintainable, and tenant-aware.
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

- Clone-first starter project with AI-facing Markdown instructions.
- Canonical strict JSON resource definitions under `resources/*.resource.json`.
- Strict JSON policy bindings under `policies/*.policy.json`.
- PostgreSQL SQL migration generation with destructive-change approval plans.
- Basic HTTP CRUD API generation.
- Minimal approved tagged-template custom SQL helper and generated query helpers that parameterize values and allow-list identifiers.
- Tenant-scoped query enforcement.
- Structured logging.

### Phase 2: Secure identity and policy engine

- Upstream trusted identity / no-login mode for deployments protected by Cloudflare Access or an equivalent external access layer.
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

- Supply-chain security checks using pinned `pnpm`, frozen lockfiles, CycloneDX SBOM generation, OSV-Scanner, and package-manager native audit signals.
- Release signing and provenance.
- SBOM publishing.
- Performance benchmarks.
- Multi-tenant migration tooling.
- Operational runbooks.

## Resolved design decisions

- **First resource format**: strict canonical JSON under `resources/*.resource.json`, validated by JSON Schema, is the Phase 1 source of truth. Generated SQL, TypeScript model types, OpenAPI, and JSON Schema are outputs.
- **First policy format**: strict Daicho-owned JSON policy bindings under `policies/*.policy.json`, with a small expression vocabulary and deny-by-default missing/unsupported behavior.
- **SQL and migrations**: generated or reviewed SQL is the human-facing migration artifact. Custom SQL must use an approved tagged-template helper; destructive migrations require checksum-bound human approval. Kysely is the first TypeScript query-builder candidate to evaluate for richer composition.
- **Package and runtime baseline**: use Corepack-pinned `pnpm`, Node.js `>=24 <25`, and PostgreSQL `>=17 <19` with PostgreSQL 18 preferred.
- **SBOM and vulnerability scanning**: generate CycloneDX JSON with `@cyclonedx/cdxgen`, scan with OSV-Scanner, and also run package-manager native audit signals where available.
- **PostgreSQL row-level security**: RLS generation is not required for the first prototype. It should remain a future optional capability.
- **First identity mode**: implement a no-login / upstream trusted identity mode first, assuming deployments can be protected by Cloudflare Access or an equivalent external access layer. OIDC, SAML, and passkeys can follow as explicit adapters.
- **Minimum viable deployment target**: provide Docker Compose or an equivalent local container setup first so teams can test Daicho easily.
- **Optional UI generation**: Daicho itself may not need a management UI. Generated applications should be manageable through APIs, CLIs, and application-specific workflows; any optional UI generator should be separate from the core runtime unless a strong reason emerges.

## Contributing

This repository is currently in the design stage. Contributions should focus on improving the specification, identifying security risks, validating deployment assumptions, and proposing minimal implementation paths.

Please keep proposals text-first, auditable, and compatible with AI-agent-assisted development workflows.
