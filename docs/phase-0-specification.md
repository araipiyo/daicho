# Daicho Phase 0 Specification

Status: Draft 0.4
Phase: 0

## 1. Purpose

Phase 0 produces the smallest specification package needed to start the Phase 1 prototype safely.

## 2. Phase 0 outcome

Phase 0 is complete when maintainers have accepted:

- Product scope and non-goals aligned to the enterprise foundation vision: SSO, SCIM-ready provisioning, audit, granular permissions, secure defaults, PostgreSQL, and declarative CRUD.
- Runtime-library-first architecture.
- AI-agent-first, command-capable developer workflow.
- Web application and web API product-user delivery model.
- Clone-first starter workflow.
- Phase 1 CLI command set.
- Resource and tenancy model.
- Deny-by-default authorization model.
- Identity lifecycle model for active, disabled, suspended, and deprovisioned principals.
- Audit requirements.
- SQL restrictions.
- Deployment preparation and protected-ingress model.
- Runtime linkage and update model.
- Resource authoring format.
- Policy authoring model.
- Custom SQL escape hatch.
- Package manager, runtime/database version ranges, SBOM, and vulnerability scanning tools.
- Destructive migration approval workflow.
- Prototype acceptance scenario.

## 3. Required documents

The Phase 0 package consists of:

- `docs/specification.md`.
- `docs/architecture.md`.
- `docs/security.md`.
- `docs/threat-model.md`.
- `docs/resource-model.md`.
- `docs/cli.md`.
- `docs/prototype-acceptance.md`.
- `docs/decisions.md`.

Each document should stay short and contain only implementation, review, or security-relevant detail.

## 4. Phase 1 assumptions

Phase 1 assumes:

- Developers start by cloning a maintained starter repo and commonly use AI coding agents such as Codex or Claude Code.
- The CLI validates, tests, exports, prepares deployment, and diagnoses problems for developers, AI agents, CI, and limited operator workflows.
- Product users consume the deliverable as a web application, web API, service integration, or background workflow, not through the Daicho CLI.
- The runtime library enforces security invariants.
- PostgreSQL is the system of record.
- Upstream trusted identity is the first authentication mode, with OIDC, SAML, passkeys, service credentials, and SCIM provisioning preserved as explicit adapter boundaries.
- Password login and admin UI generation are absent by default.
- Deployment is prepared for humans, not silently applied.
- Non-local deployments prefer protected ingress and avoid direct public origin exposure.
- Phase 1 may colocate the runtime in the starter repo, but the runtime linkage boundary must be explicit.

## 5. Phase 1 command set

Required commands:

- `daicho check`.
- `daicho test`.
- `daicho export`.
- `daicho deploy prepare`.
- `daicho doctor`.

Not required:

- `daicho init`.
- General scaffolding.
- Full code generation.
- Automatic cloud deployment.
- Product-user CLI requirements.

## 6. Enforced security rules

Phase 1 must enforce or test:

- Missing principal is rejected.
- Disabled, suspended, or deprovisioned principals are rejected.
- Tenant-scoped resources require tenant context.
- Cross-tenant access fails.
- Missing policy denies access.
- Resource database access uses approved helpers.
- Raw SQL escape hatches are disabled by default.
- Writes produce audit events.
- Plans and logs redact secrets.
- Default starter excludes password login.
- Upstream trusted identity rejects unverified ingress and spoofable headers.
- Deployment guidance discourages direct public origin exposure.
- Runtime linkage reports an exact version or commit.
- Resource files validate against strict JSON Schema before generated artifacts are used.
- Policy files validate against the Phase 1 JSON policy schema and deny unsupported expressions.
- Destructive migration plans require explicit human approval tied to migration checksums.
- SBOM and vulnerability scans run in CI with documented exception handling.
- Product users can access the delivered web app or API without the Daicho CLI.

If a rule is not enforced at runtime, Phase 0 must name the test, lint rule, or review gate that enforces it.

## 7. Review workflow

Before Phase 1 starts:

1. Review each required document for contradictions.
2. Record accepted decisions in `docs/decisions.md`.
3. Mark unresolved questions as deferred or blocking.
4. Confirm the prototype acceptance checklist.
5. Keep only details that affect implementation, review, or security.

## 8. Deferred questions

Phase 0 may defer:

- Row-level security roadmap.
- Provider-specific protected-ingress adapters and proof formats.
- Post-Phase 1 runtime distribution mechanism.
