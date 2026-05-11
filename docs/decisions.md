# Daicho Decision Log

Status: Draft 0.3
Phase: 0

## 1. Purpose

This document records decisions that shape Daicho implementation.

## 2. Decision format

Each full decision record should include:

- Identifier.
- Title.
- Status.
- Context.
- Decision.
- Consequences.
- Alternatives considered.
- Date accepted.
- Owners or reviewers.

## 3. Accepted decisions

### D0001: AI-agent-first, command-capable development

Status: Accepted
Decision: Daicho must be usable through source files, AI coding agents such as Codex or Claude Code, CLI commands, tests, and CI/CD without a development GUI.
Consequence: CLI behavior, generated artifacts, deterministic plans, and human-reviewable diffs are core developer surfaces.

### D0002: No default administrative GUI or product-user CLI

Status: Accepted
Decision: Daicho must not generate or require an administrative GUI by default, and product users must not be required to run the Daicho CLI.
Consequence: Product users consume web applications, web APIs, service integrations, or background workflows; operators and developers use APIs, CLIs, auditable workflows, and optional separately maintained UIs.

### D0003: PostgreSQL as system of record

Status: Accepted
Decision: PostgreSQL is the first transactional database.
Consequence: Migrations, tenancy, audit, and query safety focus on PostgreSQL first.

### D0004: Upstream trusted identity first

Status: Accepted
Decision: The first identity mode is no-login / upstream trusted identity.
Consequence: Password login is excluded from the default starter; OIDC, SAML, passkeys, service credentials, and passwords remain explicit adapters.

### D0005: Deny-by-default authorization

Status: Accepted
Decision: Resource operations deny by default unless policies allow them.
Consequence: Missing policies fail validation or deny access with auditable output.

### D0006: Runtime-library-first enforcement

Status: Accepted
Decision: Security-critical behavior belongs in the Daicho runtime library and approved helpers, not in copied application boilerplate.
Consequence: Starter code stays thin, and tests focus on framework-enforced invariants.

### D0007: Clone-first Phase 1 starter

Status: Accepted
Decision: Phase 1 starts from a maintained starter repo, not a project generator.
Consequence: `daicho init` and broad scaffolding are deferred.

### D0008: Protected ingress by default

Status: Accepted
Decision: Daicho deliverables should run behind a protected access layer such as Cloudflare Access, Tailscale, identity-aware proxies, API gateways, private service meshes, or equivalent zero-trust products. Direct public origin exposure is discouraged even for public applications unless explicitly documented with compensating controls.
Consequence: Deployment templates, checks, and runtime adapters must model ingress trust, reject spoofable identity headers, and distinguish verified ingress from trusted-network-only and unverified ingress.

### D0009: Runtime linkage must be explicit

Status: Accepted
Decision: Phase 1 may keep the Daicho runtime in the clone-first starter repository, but runtime imports, ownership boundaries, version identity, and update paths must be explicit. The preferred future model is a separately versioned, immutable, provenance-checked runtime dependency.
Consequence: Starter layout and checks must prevent copied security-critical internals from becoming invisible forks, and future distribution work must support security patches across applications.

## 4. Pending Phase 0 decisions

- D0101: First resource authoring format.
- D0102: Generated or starter code ownership and overwrite model.
- D0103: First policy authoring model.
- D0104: Minimum safe custom SQL escape hatch.
- D0105: First package manager.
- D0106: Node.js and PostgreSQL version ranges.
- D0107: SBOM and vulnerability scanning tools.
- D0108: Destructive migration approval workflow.
- D0109: Row-level security roadmap.
- D0110: Provider-specific protected-ingress adapters and proof formats.
- D0111: Post-Phase 1 runtime distribution mechanism.
