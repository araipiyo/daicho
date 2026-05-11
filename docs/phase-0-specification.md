# Daicho Phase 0 Specification

Status: Draft 0.2
Phase: 0 - specification only

Phase 0 defines the smallest set of decisions needed before implementation starts. It is not an ISO-style specification. It should stay short, readable, and easy to audit.

## 1. Goal

Daicho should help teams build AI-written operational applications that are safe enough to review and deploy.

Phase 0 is complete when maintainers can answer these questions without guessing:

- What is the first usable Daicho project?
- Which security rules are enforced by the framework itself?
- What must humans review before deployment?
- What is intentionally not included yet?

## 2. First user experience

The first development workflow should be simple:

1. `git clone` a starter repository.
2. Edit resource, policy, and deployment files in plain text.
3. Run tests and validation from the command line.
4. Deploy using reviewed templates.

A project generator is not required for Phase 1. A generator may be added later only if it removes real maintenance work without hiding important security or deployment choices.

## 3. Phase 1 product boundary

Phase 1 should build one narrow prototype:

- TypeScript application code.
- PostgreSQL as the transactional database.
- One starter repository that can be cloned.
- One tenant-scoped resource example.
- CRUD HTTP API for that resource.
- Upstream trusted identity mode for local and deployed use.
- Deny-by-default authorization.
- Tenant isolation checks.
- Audit events for writes and denied access.
- OpenAPI and JSON Schema export.
- Docker Compose deployment template.

Anything outside this list is optional unless a maintainer explicitly accepts it into Phase 1.

## 4. Architecture decision

Daicho should prefer a small enforced runtime library over large generated security-sensitive code.

The application should link to Daicho framework packages that own these checks:

- authentication context parsing;
- tenant context validation;
- authorization dispatch;
- safe database access helpers;
- audit event writing;
- request and response validation hooks;
- startup configuration validation.

AI agents may write application code, resources, policies, tests, and adapters, but they must not be trusted to remember security rules. Security-critical behavior should fail closed inside the Daicho library when required context, policy, tenant scope, or audit configuration is missing.

Generated or template code may exist, but it should stay thin and reviewable. It should call the framework library instead of duplicating security logic.

## 5. CLI decision

Phase 1 needs command-line checks more than a full generator.

Required commands:

- `daicho check`: validate resources, policies, tenancy declarations, configuration, and dependency risk.
- `daicho test`: run the project test suite and Daicho security checks.
- `daicho export`: write OpenAPI and JSON Schema artifacts.
- `daicho deploy prepare`: prepare deployment files, plans, and human-readable instructions without applying infrastructure.
- `daicho doctor`: report local environment and configuration problems.

Not required for Phase 1:

- `daicho init`;
- general project scaffolding;
- full application code generation;
- automatic cloud deployment.

Commands that can affect deployment or data must have dry-run or plan output. Plans must not contain secrets.

## 6. Deployment decision

Deployment is likely to be performed by a human, so Phase 1 should provide templates and checks rather than hidden automation.

The first deployment support should include:

- Docker Compose for local and simple server deployment;
- documented environment variables;
- health and readiness endpoints;
- migration instructions;
- reverse-proxy or upstream identity assumptions;
- a deployment checklist for humans;
- `daicho deploy prepare` output showing what will be used.

Daicho must not require a proprietary hosted control plane.

## 7. Security decisions

Security must be enforced, not merely suggested.

Phase 1 must enforce these rules in framework code or tests:

- Requests without a valid principal are rejected.
- Tenant-scoped resources require an explicit tenant context.
- Cross-tenant reads and writes fail.
- Missing authorization policy means deny.
- Database access for resources goes through Daicho-approved helpers.
- Raw SQL escape hatches are disabled by default.
- Writes produce audit events.
- Secrets are never printed in plans or logs.
- Password login is not included in the starter.

If a rule cannot be enforced in runtime code, Phase 0 must name the test, lint rule, or review gate that enforces it.

## 8. Required Phase 0 documents

Keep the document set small. Before Phase 1 starts, these files should exist and agree with each other:

- `docs/specification.md`: product scope and non-goals.
- `docs/architecture.md`: runtime-library architecture and boundaries.
- `docs/security.md`: enforced security invariants.
- `docs/threat-model.md`: realistic threats and mitigations.
- `docs/resource-model.md`: resource shape and tenancy model.
- `docs/cli.md`: required commands and output behavior.
- `docs/prototype-acceptance.md`: end-to-end acceptance checklist.
- `docs/decisions.md`: accepted and deferred decisions.

Each document should be short. Details belong only where they change implementation, review, or security decisions.

## 9. Acceptance checklist

Phase 0 can close when maintainers have accepted:

- [ ] the clone-first starter workflow;
- [ ] no Phase 1 project generator;
- [ ] the required Phase 1 CLI commands;
- [ ] the runtime-library enforcement model;
- [ ] tenant isolation invariants;
- [ ] authorization deny-by-default behavior;
- [ ] audit requirements;
- [ ] raw SQL restrictions;
- [ ] deployment templates and human checklist;
- [ ] the single end-to-end prototype scenario.

## 10. Deferred questions

These questions may be deferred if they do not block the prototype:

- Should a generator be added after Phase 1?
- Which cloud-specific deployment templates should come first?
- How much PostgreSQL row-level security should Daicho manage directly?
- What is the safest limited raw SQL extension model?
- Which authentication adapters should follow upstream trusted identity mode?
- Which package manager, SBOM tool, and vulnerability scanner should be recommended by default?
