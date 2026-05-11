# Daicho Framework Specification

Status: Draft 0.2
Phase: 0

## 1. Purpose

Daicho is a security-first, text-first TypeScript framework for AI-era operational systems and CRUD APIs. It replaces GUI-first admin tooling with reviewable source files, CLI checks, generated artifacts, tests, and CI/CD.

Phase 0 defines the implementation-ready specification set. Phase 1 proves the first prototype.

## 2. Core principles

Daicho must:

- Be usable without a development GUI or default administrative GUI.
- Be secure by default and deny by default.
- Disable password login by default.
- Support upstream trusted identity / no-login mode first.
- Use PostgreSQL as the first transactional database.
- Treat tenancy, policy, audit, and SQL safety as framework invariants.
- Emit deterministic, reviewable source, SQL, OpenAPI, JSON Schema, plans, and diagnostics.
- Avoid a proprietary hosted control plane requirement.
- Keep dependencies minimal, pinned, reviewed, and replaceable.

## 3. Required product surface

Phase 1 must provide:

- A clone-first starter project.
- A runtime library that enforces identity, tenancy, policy, SQL safety, and audit rules.
- A CLI for validation, tests, exports, deployment preparation, and diagnostics.
- PostgreSQL migrations committed as source.
- OpenAPI and JSON Schema exports.
- Docker Compose deployment support and a human deployment checklist.
- Structured logs, health checks, readiness checks, and redacted plans.

Phase 1 does not require:

- A project generator.
- Full application code generation.
- Built-in admin UI generation.
- Automatic cloud deployment.
- Built-in password authentication.

## 4. Required CLI commands

Phase 1 commands:

- `daicho check`: validate resources, policies, tenancy, configuration, and dependency risk.
- `daicho test`: run project tests and Daicho security checks.
- `daicho export`: write OpenAPI and JSON Schema artifacts.
- `daicho deploy prepare`: prepare deployment files, plans, and instructions without applying infrastructure.
- `daicho doctor`: report local environment and configuration problems.

Commands that can affect data or deployment must support dry-run or plan output. Plans must not contain secrets.

## 5. Runtime requirements

The runtime must:

- Reject requests without a valid principal.
- Require tenant context for tenant-scoped resources.
- Evaluate policy before mutation or broad data access.
- Deny missing policies.
- Prevent cross-tenant reads and writes by construction.
- Route resource database access through Daicho-approved helpers.
- Disable raw SQL escape hatches by default.
- Write audit events for security-relevant operations.
- Redact secrets from logs, plans, errors, examples, and manifests.

## 6. Resource requirements

Every resource must declare:

- Stable name.
- Tenancy mode: `global`, `tenant_scoped`, `tenant_partitioned`, or `system`.
- Database mapping.
- Fields and validation.
- Indexes and relationships.
- Exposed operations.
- Policy bindings.
- Audit behavior.

Tenant-scoped resources must include tenant constraints in generated or framework-provided queries.

## 7. Phase 0 document set

The specification set is:

- `docs/specification.md`: product scope and non-goals.
- `docs/architecture.md`: runtime-library architecture and boundaries.
- `docs/security.md`: enforced security invariants.
- `docs/threat-model.md`: threats and controls.
- `docs/resource-model.md`: resource shape and tenancy model.
- `docs/cli.md`: required commands and output behavior.
- `docs/prototype-acceptance.md`: end-to-end acceptance checklist.
- `docs/decisions.md`: accepted and deferred decisions.

## 8. Phase 0 acceptance

Phase 0 can close when maintainers accept:

- Clone-first starter workflow.
- No Phase 1 project generator.
- Phase 1 CLI command set.
- Runtime-library enforcement model.
- Tenant isolation invariants.
- Deny-by-default authorization.
- Audit requirements.
- Raw SQL restrictions.
- Deployment templates and human checklist.
- Single end-to-end prototype scenario.

## 9. Deferred questions

These may wait until after the prototype if documented:

- Post-Phase 1 generator scope.
- Cloud-specific deployment templates.
- PostgreSQL row-level security automation.
- Safe custom SQL extension model.
- Adapter order after upstream trusted identity.
- Default package manager, SBOM tool, and vulnerability scanner.
