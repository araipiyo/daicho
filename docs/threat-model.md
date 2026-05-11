# Daicho Threat Model

Status: Draft 0.1  
Phase: 0

## 1. Purpose

This document identifies threats that Daicho must consider before implementation. It focuses on the first prototype and security-critical design choices.

## 2. Assets

Daicho must protect:

- Tenant business data.
- Principal identities and service credentials.
- Authorization policies.
- Audit records.
- Migration files.
- Generated source code.
- Deployment manifests.
- Plans and diagnostic outputs.
- Package and template integrity.

## 3. Actors

Relevant actors include:

- Legitimate end users.
- Tenant administrators.
- System operators.
- External attackers.
- Malicious or compromised dependencies.
- Misconfigured identity providers.
- Compromised CI/CD systems.
- AI coding agents operating with repository or shell access.

## 4. Trust boundaries

Important trust boundaries are:

- Client to upstream identity boundary.
- Upstream identity boundary to generated application.
- Generated application to PostgreSQL.
- Generated application to optional adapters.
- Developer workstation to package registry.
- CI/CD to deployment platform.
- Daicho templates to generated user projects.

## 5. Priority threats

| Threat | Impact | Required controls |
| --- | --- | --- |
| Cross-tenant data access | Critical | Tenant-aware generated queries, policy tests, tenant constraints in repositories |
| SQL injection | Critical | Parameter binding, safe identifier handling, restricted raw SQL escape hatches |
| Authorization bypass | Critical | Deny-by-default policies, generated policy calls, negative tests |
| Spoofed upstream identity headers | High | Trusted boundary configuration, header validation, deployment guidance |
| Secret leakage in plans or logs | High | Redaction, structured logging rules, tests |
| Compromised dependency | High | Minimal dependencies, lockfiles, vulnerability checks, SBOM path |
| Template tampering | High | Review process, template integrity checks, generated-file provenance |
| Unsafe migrations | High | Plans, destructive-change warnings, manual approvals |
| Audit tampering or loss | High | Append-only application behavior, audit failure policy, restricted writes |
| AI-agent mistakes | Medium/High | Plan-before-apply workflow, code review, deterministic outputs, tests |

## 6. Validation strategy

Each high-priority threat must map to at least one validation method:

- Unit tests for policy decisions.
- Integration tests for tenant isolation.
- SQL generation tests for parameter binding.
- Snapshot tests for redacted plans.
- CLI tests for destructive-change warnings.
- Dependency checks in CI.
- Documentation review for deployment trust boundaries.

## 7. Residual risks for first prototype

The first prototype may defer:

- Full PostgreSQL row-level security generation.
- Production-ready OIDC, SAML, and passkey adapters.
- Multi-database tenant partitioning.
- Formal verification of policy logic.

These deferrals are acceptable only if the prototype remains local/development-focused and documents the limits clearly.
