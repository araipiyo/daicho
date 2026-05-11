# Daicho Threat Model

Status: Draft 0.2
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

## 3. Actors

Relevant actors are legitimate users, tenant administrators, operators, external attackers, compromised dependencies, misconfigured identity providers, compromised CI/CD systems, and AI coding agents with repository or shell access.

## 4. Trust boundaries

Important boundaries are:

- Client to upstream identity layer.
- Upstream identity layer to application.
- Application to PostgreSQL.
- Application to optional adapters.
- Developer workstation to package registry.
- CI/CD to deployment platform.
- Daicho templates to user projects.

## 5. Priority threats and controls

| Threat | Impact | Required controls |
| --- | --- | --- |
| Cross-tenant data access | Critical | Tenant-aware helpers, policy tests, repository constraints |
| SQL injection | Critical | Parameter binding, safe identifiers, restricted raw SQL |
| Authorization bypass | Critical | Deny-by-default policies, generated policy calls, negative tests |
| Spoofed upstream identity headers | High | Trusted-boundary config, header validation, deployment guidance |
| Secret leakage | High | Redaction rules, structured logging, snapshot tests |
| Compromised dependency | High | Minimal dependencies, lockfiles, vulnerability checks, SBOM path |
| Template tampering | High | Review process, integrity checks, provenance markers |
| Unsafe migrations | High | Plans, destructive-change warnings, manual approval |
| Audit tampering or loss | High | Restricted writes, audit failure policy, append-only behavior |
| AI-agent mistakes | Medium/High | Plan-before-apply workflow, review, deterministic output, tests |

## 6. Validation strategy

High-priority threats must map to tests or review gates:

- Unit tests for policy decisions.
- Integration tests for tenant isolation.
- SQL tests for parameter binding.
- Snapshot tests for redacted plans.
- CLI tests for destructive-change warnings.
- Dependency checks in CI.
- Documentation review for trust boundaries.

## 7. Deferred risks

The first prototype may defer:

- Full PostgreSQL row-level security generation.
- Production OIDC, SAML, and passkey adapters.
- Multi-database tenant partitioning.
- Formal verification of policy logic.

Deferrals are acceptable only when documented and when the prototype remains clear about its limits.
