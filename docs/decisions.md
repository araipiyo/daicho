# Daicho Decision Log

Status: Draft 0.1  
Phase: 0

## 1. Purpose

This document records accepted and pending decisions that shape Daicho implementation.

## 2. Decision format

Each decision should include:

- Identifier.
- Title.
- Status.
- Context.
- Decision.
- Consequences.
- Alternatives considered.
- Date accepted.
- Owners or reviewers.

## 3. Accepted baseline decisions

### D0001: Command-first development

Status: Accepted  
Decision: Daicho must be usable through source files, CLI commands, tests, and CI/CD without a development GUI.  
Consequence: CLI behavior and generated artifacts are core product surfaces.

### D0002: No default administrative GUI

Status: Accepted  
Decision: Daicho must not generate or require an administrative GUI by default.  
Consequence: Operators use APIs, CLIs, auditable workflows, and optional separately maintained UIs.

### D0003: PostgreSQL as system of record

Status: Accepted  
Decision: PostgreSQL is the primary transactional database for the first prototype.  
Consequence: Migrations, tenancy, audit, and query safety focus on PostgreSQL first.

### D0004: Upstream trusted identity first

Status: Accepted  
Decision: The first identity mode should be no-login / upstream trusted identity.  
Consequence: Password login is excluded from the default template, and OIDC, SAML, passkeys, and passwords remain explicit adapters.

### D0005: Deny-by-default authorization

Status: Accepted  
Decision: Resource operations must deny by default unless policies allow them.  
Consequence: Missing policies must fail validation or deny access with auditable output.

## 4. Pending Phase 0 decisions

- D0101: First resource authoring format.
- D0102: Generated code ownership and overwrite model.
- D0103: First policy authoring model.
- D0104: Minimum safe custom SQL escape hatch.
- D0105: First package manager for generated projects.
- D0106: Node.js and PostgreSQL version ranges.
- D0107: SBOM and vulnerability scanning tools.
- D0108: Destructive migration approval workflow.
- D0109: Row-level security roadmap.
