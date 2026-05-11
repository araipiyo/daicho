# Daicho Security Specification

Status: Draft 0.4
Phase: 0

## 1. Purpose

This document defines mandatory security requirements for Daicho specs, runtime code, CLI checks, starter code, deployment templates, protected ingress, and runtime linkage.

## 2. Security posture

Daicho must be secure by default. Convenience must not weaken authentication, authorization, tenancy, SQL safety, auditability, protected ingress, secret handling, runtime updateability, or supply-chain controls.

## 3. Default project requirements

Default projects must:

- Avoid password login.
- Avoid administrative GUIs.
- Require a principal for requests.
- Deny resource operations unless policy allows them.
- Require every resource to declare tenancy.
- Generate or use tenant-safe query helpers for tenant-scoped resources.
- Use parameterized SQL or an equivalent safe query builder.
- Validate canonical resource JSON and policy JSON before generation, export, migration planning, or deployment preparation.
- Redact secrets from logs, plans, errors, and manifests.
- Include command-line security checks for developers, AI agents, CI, and operators.
- Expose product functionality through web applications, web APIs, or service interfaces rather than requiring product users to run the Daicho CLI.
- Declare the expected protected-ingress mode for non-local deployments.

## 4. Development and user security model

AI coding agents are expected development actors and must be treated as capable of making unsafe changes unless constrained by reviewable plans, tests, deterministic output, source control, and CI gates. Daicho workflows should make AI-agent output easy to review and should prefer plan-before-apply commands for data, migrations, and deployment.

The Daicho CLI is not a product-user interface. Product users must access deliverables through web applications, web APIs, service integrations, or background workflows with normal application authentication and authorization controls.

## 5. Authentication

The first identity mode is upstream trusted identity / no-login mode. It may trust identity headers only behind an explicitly configured trusted boundary and should prefer cryptographically verifiable access-layer evidence where available.

The mode must define:

- Accepted headers.
- Local development simulation.
- Trusted proxy or access-layer assumptions.
- Whether ingress is verified, trusted-network-only, or unverified.
- Header-spoofing controls.
- Behavior for absent or malformed identity.

OIDC, SAML, passkeys, service credentials, protected-ingress providers, and passwords are explicit adapters. Password authentication remains opt-in if implemented.

## 6. Authorization

Authorization is deny-by-default. Handlers must call policy checks before mutations and before broad data access.

Policies receive documented inputs and return:

- Decision: allow or deny.
- Safe reason code.
- Optional safe explanation.
- Audit metadata.

Denied requests must not reveal sensitive record existence across tenant boundaries.

## 7. Tenancy

Every resource must use exactly one tenancy mode: `global`, `tenant_scoped`, `tenant_partitioned`, or `system`.

For `tenant_scoped` resources, tenant constraints must apply to reads, lists, updates, deletes, relationship traversal, uniqueness checks, and existing-record lookups.

## 8. SQL safety

Generated and starter code must not interpolate untrusted values into SQL strings. Values use parameter binding. Identifiers use generated or allow-listed safe handling.

Custom SQL escape hatches are disabled by default. The minimum approved escape hatch is a tagged-template helper that parameterizes values and only accepts generated or allow-listed identifier tokens. It may concatenate only already-tokenized SQL fragments. Each use must be explicit, searchable, tested, reviewed for tenant constraints, and absent from default paths. Kysely is the preferred TypeScript query-builder candidate for future richer SQL composition; generated or reviewed SQL remains the human-facing migration artifact for Phase 1.

## 9. Protected ingress and origin exposure

Production deployments should put Daicho applications behind Cloudflare Access, Tailscale, an identity-aware proxy, API gateway, private service mesh, or an equivalent zero-trust access layer. Direct public exposure of the origin server is not recommended, including for public applications, unless maintainers document the exception and compensating controls.

Required controls for upstream trusted identity mode:

- Identity headers must be accepted only from configured trusted boundaries.
- Signed access-layer JWTs, mTLS client certificates, signed service tokens, or equivalent provider proofs should be validated when available.
- Network-only trust must be paired with private networking, firewall allow-lists, tunnel configuration, or equivalent controls that prevent arbitrary clients from reaching the origin.
- Unverified ingress must fail checks for upstream trusted identity mode.
- Health and readiness endpoints must avoid exposing sensitive details and must not bypass ingress assumptions for non-local deployments.

Daicho checks may inspect configuration and runtime adapters for declared ingress proof, but the framework cannot prove an access path from plain unsigned headers alone.

## 10. Secrets and redaction

Secrets must not appear in:

- Plans.
- Logs.
- Errors.
- OpenAPI.
- JSON Schema.
- Examples.
- Committed deployment manifests.

If a secret affects review, output a redacted placeholder and safe metadata such as variable name or source name.

## 11. Audit

Applications must write audit events for security-relevant operations. Audit records should include event ID, timestamp, correlation ID, principal ID, tenant ID when applicable, resource, operation, result, safe reason code, and redacted change summary.

Audit persistence failures during mutations must have a specified failure policy. The default should fail closed for security-sensitive mutations.

## 12. Migration safety

Resource changes must produce a migration plan before any database change is applied. Plans must classify each change as safe additive, potentially destructive, rename-like, or manual. Potentially destructive and manual changes require explicit human approval tied to the migration checksum; AI agents may prepare the plan but must not auto-approve it.

The migration runner must record applied migration names and checksums in PostgreSQL and refuse to apply a migration whose committed checksum differs from the recorded checksum unless a documented repair procedure is followed.

## 13. Supply-chain security

Phase 1 projects use `pnpm` with an exact Corepack-pinned package-manager version and a committed lockfile. CI must install with a frozen lockfile. Lifecycle scripts should be disabled for security checks where practical, and required scripts must be documented and allow-listed.

Projects must generate CycloneDX JSON SBOMs with `@cyclonedx/cdxgen`, scan lockfiles and SBOMs with OSV-Scanner, and run the package-manager native audit path where available. Known critical vulnerabilities fail CI unless a reviewed, time-limited exception is committed.

## 14. Runtime and database security baseline

Phase 1 production and CI use Node.js `>=24 <25`. PostgreSQL `>=17 <19` is supported, with PostgreSQL 18 preferred and PostgreSQL 17 accepted for managed-provider compatibility. Deployments must track the latest security patch or minor release for the selected major line.

## 15. Runtime linkage and update security

Phase 1 may place the Daicho runtime in the clone-first starter repository, but projects must keep a clear boundary between framework-owned runtime code and application-owned code. Security-critical runtime internals must be imported through stable APIs rather than copied into application code.

The preferred future model is a separately versioned runtime dependency with immutable pins, provenance, integrity verification, and a documented security update and rollback workflow. Local runtime forks or modifications should be explicit, searchable, and treated as security-sensitive.

A runtime linkage model must document how vulnerable runtime versions are detected, how patches are distributed, and how applications prove which runtime version or commit they are using.

## 16. Supply chain

Generated or starter projects must include:

- Lockfile enforcement.
- Minimal dependencies.
- Dependency review guidance.
- Vulnerability scanning path using OSV-Scanner and package-manager native audit signals.
- SBOM generation path using CycloneDX JSON.
- Release signing guidance.
- Template integrity expectations.
- Runtime version, provenance, and integrity expectations.

Dependencies affecting authentication, authorization, SQL, cryptography, or deployment require heightened review.

## 17. Acceptance

Phase 1 code must prove:

- Missing policies deny access.
- Cross-tenant reads and writes fail.
- SQL injection payloads are parameterized or rejected.
- Secrets are redacted from plans and logs.
- Password login is absent from the default template.
- Product users can use the delivered web app or API without the Daicho CLI.
- Upstream trusted identity rejects unverified ingress and spoofable identity headers.
- Deployment templates discourage direct public origin exposure.
- Runtime linkage records an exact version or commit and update path.
- Audit events are emitted for allowed and denied security-relevant operations.
- Resource and policy JSON validation fails closed for unknown keys and unsupported expressions.
- Destructive migrations require checksum-bound human approval.
- Supply-chain checks generate an SBOM and scan dependencies before release.
