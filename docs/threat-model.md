# Daicho Threat Model

Status: Draft 0.3
Phase: 0

## 1. Purpose

This document lists the threats the Phase 1 prototype must address or explicitly defer.

## 2. Assets

Daicho must protect:

- Tenant business data.
- Principal identities and service credentials.
- Authorization policies.
- Audit records.
- Migration files.
- Generated or starter source code.
- Deployment manifests.
- Plans and diagnostics.
- Package and template integrity.
- Runtime linkage integrity and security update path.
- Origin server reachability and ingress trust assumptions.

## 3. Actors

Relevant actors are product users, tenant administrators, developers, operators, AI coding agents with repository or shell access, external attackers, compromised dependencies, misconfigured identity or access-layer providers, and compromised CI/CD systems.

## 4. Trust boundaries

Important boundaries are:

- Product user to web application or web API.
- Client to upstream identity or protected access layer.
- Access layer to origin application.
- Application to PostgreSQL.
- Application to optional adapters.
- Developer workstation to package registry.
- CI/CD to deployment platform.
- Daicho templates to user projects.
- Daicho runtime artifact to application repository.

## 5. Priority threats and controls

| Threat | Impact | Required controls |
| --- | --- | --- |
| Cross-tenant data access | Critical | Tenant-aware helpers, policy tests, repository constraints |
| SQL injection | Critical | Parameter binding, safe identifiers, restricted raw SQL |
| Authorization bypass | Critical | Deny-by-default policies, generated policy calls, negative tests |
| Spoofed upstream identity headers | High | Trusted-boundary config, signed or mTLS ingress proof where available, header validation, deployment guidance |
| Secret leakage | High | Redaction rules, structured logging, snapshot tests |
| Compromised dependency | High | Minimal dependencies, lockfiles, vulnerability checks, SBOM path |
| Template tampering | High | Review process, integrity checks, provenance markers |
| Unsafe migrations | High | Plans, destructive-change warnings, manual approval |
| Audit tampering or loss | High | Restricted writes, audit failure policy, append-only behavior |
| AI-agent mistakes | Medium/High | Plan-before-apply workflow, review, deterministic output, tests |
| Direct public origin exposure | High | Protected-ingress templates, origin firewall or private network guidance, explicit exception process |
| Runtime fork or stale runtime dependency | High | Explicit runtime linkage, version pins, provenance checks, vulnerability scanning, update workflow |
| Product-user CLI dependency | Medium | Web app/API delivery requirements, CLI non-goals, acceptance tests |

## 6. Validation strategy

High-priority threats must map to tests or review gates:

- Unit tests for policy decisions.
- Integration tests for tenant isolation.
- SQL tests for parameter binding.
- Snapshot tests for redacted plans.
- CLI tests for destructive-change warnings.
- Dependency checks in CI.
- Documentation review for trust boundaries.
- Deployment checks for protected ingress and direct-origin exposure.
- Runtime linkage checks for exact version or commit identity.
- Acceptance review that product users can use the delivered web app or API without the Daicho CLI.

## 7. Deferred risks

The first prototype may defer:

- Full PostgreSQL row-level security generation.
- Production OIDC, SAML, and passkey adapters.
- Provider-specific protected-ingress adapters beyond the first upstream trusted identity proof.
- Multi-database tenant partitioning.
- Formal verification of policy logic.
- Final post-Phase 1 runtime distribution mechanism.

Deferrals are acceptable only when documented and when the prototype remains clear about its limits.
