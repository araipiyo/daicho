# Daicho Framework Specification

Status: Draft 0.3
Phase: 0

## 1. Purpose

Daicho is a security-first, text-first TypeScript framework for AI-era operational systems and CRUD APIs. It assumes developers use AI coding agents such as Codex or Claude Code with source files, reviewable diffs, CLI checks, generated artifacts, tests, and CI/CD instead of GUI-first admin tooling.

Phase 0 defines the implementation-ready specification set. Phase 1 proves the first prototype.

## 2. Core principles

Daicho must:

- Be optimized for AI-assisted development with human review of source, plans, tests, and diffs.
- Be usable without a development GUI or default administrative GUI.
- Be secure by default and deny by default.
- Disable password login by default.
- Support upstream trusted identity / no-login mode first.
- Use PostgreSQL as the first transactional database.
- Treat tenancy, policy, audit, and SQL safety as framework invariants.
- Emit deterministic, reviewable source, SQL, OpenAPI, JSON Schema, plans, and diagnostics.
- Avoid a proprietary hosted control plane requirement.
- Deliver user-facing functionality as web applications, web APIs, background jobs, or integrations; end users must not need the Daicho CLI.
- Keep dependencies minimal, pinned, reviewed, and replaceable.

## 3. Required product surface

Phase 1 must provide:

- A clone-first starter project.
- A runtime library that enforces identity, tenancy, policy, SQL safety, and audit rules.
- A CLI for developer, AI-agent, CI, and limited operator workflows such as validation, tests, exports, deployment preparation, and diagnostics.
- A web application or web API delivery path for product users.
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
- A requirement that product users install or run the Daicho CLI.

## 4. Developer, operator, and user surfaces

Developers are expected to work primarily through AI coding agents such as Codex or Claude Code, source control, code review, CI, and the Daicho CLI. The CLI is a developer, AI-agent, CI, and operator tool. Direct human CLI use should be limited to cases where it is appropriate, such as local checks, diagnostics, export generation, and deployment preparation.

Product users consume Daicho-built deliverables through web applications, web APIs, background integrations, or service-to-service interfaces. Product users must not be required to run the Daicho CLI, edit Daicho source files, or understand deployment plans.

## 5. Required CLI commands

Phase 1 commands:

- `daicho check`: validate resources, policies, tenancy, configuration, and dependency risk.
- `daicho test`: run project tests and Daicho security checks.
- `daicho export`: write OpenAPI and JSON Schema artifacts.
- `daicho deploy prepare`: prepare deployment files, plans, and instructions without applying infrastructure.
- `daicho doctor`: report local environment and configuration problems.

Commands that can affect data or deployment must support dry-run or plan output. Plans must not contain secrets.

## 6. Runtime requirements

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

## 7. Resource requirements

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

## 8. Deployment and access model

Daicho deliverables should be deployed behind an explicit access layer such as Cloudflare Access, Tailscale, an identity-aware proxy, API gateway, private service mesh, or equivalent zero-trust product. Even when an application is intended for broad public use, the origin server should not be exposed directly to the public Internet unless there is an explicit documented exception and compensating controls.

Recommended deployment posture:

- Bind the origin to a private network, loopback interface, private load balancer, tunnel, or firewall rule that only accepts traffic from the configured access layer.
- Prefer upstream trusted identity, OIDC, SAML, passkeys, mTLS, or signed service tokens over in-application passwords.
- Reject identity headers unless they arrive through a configured trusted boundary.
- Require TLS at the user-facing edge and authenticated, encrypted transport from the edge to the origin where supported.
- Include health and readiness endpoints that do not reveal sensitive application state.

The framework can help confirm the access path only when the access layer provides verifiable evidence, such as signed JWTs, mTLS client certificates, cryptographically verifiable service tokens, trusted proxy source constraints, or provider-specific headers validated against a trusted issuer. Plain headers alone are not proof of the path. Daicho checks and runtime adapters should therefore distinguish between:

- **Verified ingress**: the runtime validates a signed token, mTLS identity, or equivalent cryptographic proof from the access layer.
- **Trusted-network ingress**: deployment configuration restricts traffic to known proxies or private networks, and the runtime accepts configured headers only from that boundary.
- **Unverified ingress**: no reliable proof exists; the runtime must not trust identity headers and should fail security checks for upstream trusted identity mode.

## 9. Runtime linkage model

Phase 1 uses a clone-first starter workflow, so the Daicho runtime may initially live in the same repository as the application template for simplicity and reviewability. The starter must still make runtime boundaries explicit: application code imports runtime APIs through stable package names or paths and must not copy security-critical runtime internals.

The long-term preferred model is to link the application to a separately versioned Daicho runtime artifact, such as a pinned package, signed release archive, Git submodule, or other immutable dependency with provenance, integrity checks, and an update policy. Separating the runtime from each application is expected to improve security updates, vulnerability response, provenance review, and avoidance of unreviewed local modifications.

Any runtime linkage model must specify:

- Exact version or commit identity.
- Integrity or provenance verification where available.
- Update and rollback workflow.
- Whether local runtime modifications are allowed.
- How security patches reach existing applications.
- Which files are application-owned versus framework-owned.

## 10. Phase 0 document set

The specification set is:

- `docs/specification.md`: product scope and non-goals.
- `docs/architecture.md`: runtime-library architecture and boundaries.
- `docs/security.md`: enforced security invariants.
- `docs/threat-model.md`: threats and controls.
- `docs/resource-model.md`: resource shape and tenancy model.
- `docs/cli.md`: required commands and output behavior.
- `docs/prototype-acceptance.md`: end-to-end acceptance checklist.
- `docs/decisions.md`: accepted and deferred decisions.

## 11. Phase 0 acceptance

Phase 0 can close when maintainers accept:

- Clone-first starter workflow.
- No Phase 1 project generator.
- Phase 1 CLI command set.
- Runtime-library enforcement model.
- Tenant isolation invariants.
- Deny-by-default authorization.
- Audit requirements.
- Raw SQL restrictions.
- Deployment templates, protected-origin guidance, and human checklist.
- Developer CLI scope and product-user web/API scope.
- Runtime linkage model.
- Single end-to-end prototype scenario.

## 12. Deferred questions

These may wait until after the prototype if documented:

- Post-Phase 1 generator scope.
- Cloud-specific deployment templates.
- Provider-specific ingress verification adapters.
- PostgreSQL row-level security automation.
- Safe custom SQL extension model.
- Adapter order after upstream trusted identity.
- Default package manager, SBOM tool, and vulnerability scanner.
- Final post-Phase 1 runtime distribution and linkage mechanism.
