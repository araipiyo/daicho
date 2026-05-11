# Daicho Phase 0 Specification Package

Status: Draft 0.1  
Phase: 0 - specification, validation, and implementation readiness  
Audience: maintainers, contributors, security reviewers, implementation teams, and AI coding agents

## 1. Phase 0 purpose

Phase 0 exists to make Daicho implementation-ready before production code is written. The goal is not to build the framework yet; it is to define the contracts, threat model, architecture, acceptance criteria, and review process with enough precision that Phase 1 can begin without guessing about security-critical behavior.

Phase 0 should produce a reviewed specification package that explains what Daicho is, what it is not, which defaults are mandatory, which decisions remain open, and how future code will prove compliance with those decisions.

## 2. Phase 0 outcomes

Phase 0 is complete when the project has these reviewed artifacts:

1. **Product and scope specification**: the core problem, target users, non-goals, first prototype boundary, and adoption model.
2. **Architecture specification**: CLI, schema compiler, generator, runtime, policy engine, PostgreSQL layer, audit layer, adapter boundaries, deployment templates, and testing surfaces.
3. **Security specification**: secure-by-default rules, authentication posture, authorization model, tenant isolation model, SQL safety rules, secret handling, supply-chain expectations, and audit guarantees.
4. **Threat model**: assets, actors, trust boundaries, misuse cases, prioritized threats, required mitigations, and validation methods.
5. **Resource definition specification**: the first supported resource authoring format, required fields, static analysis requirements, validation behavior, generated artifacts, and compatibility rules.
6. **CLI command specification**: command names, required inputs, outputs, dry-run behavior, plan format, exit-code expectations, and machine-readable output rules.
7. **Prototype acceptance tests**: a testable checklist that Phase 1 code must satisfy before the prototype is considered complete.
8. **Decision log**: resolved decisions, deferred decisions, decision owners, and criteria for revisiting decisions.
9. **Contribution and review workflow**: how changes to specifications are proposed, reviewed, versioned, and translated into implementation tasks.

## 3. In-scope work

Phase 0 includes specification and design work only. In-scope activities are:

- Writing and revising Markdown specifications.
- Defining CLI behavior, plan schemas, generated artifact expectations, and failure modes.
- Defining security invariants and acceptance tests that future implementation must satisfy.
- Creating example resource definitions and generated-output sketches when they clarify behavior.
- Identifying mandatory dependencies, forbidden dependency categories, and review requirements.
- Defining documentation structure and naming conventions.
- Creating issue-ready implementation tasks for Phase 1.

## 4. Out-of-scope work

Phase 0 must not drift into implementation. The following are out of scope unless explicitly reclassified by a maintainer decision:

- Building the Daicho CLI.
- Implementing the runtime server.
- Generating real migrations or application source code.
- Shipping authentication adapters.
- Creating an administrative GUI or development GUI.
- Selecting a long-term cloud hosting provider as a required control plane.
- Adding a password-login starter template.
- Publishing packages to npm.

Small illustrative snippets are allowed when they document intended behavior, but they must be labeled as examples and must not be treated as production code.

## 5. Guiding principles for all Phase 0 documents

Every Phase 0 document must follow these principles:

- **English-first**: all normative specification text must be written in English.
- **Normative clarity**: use `must`, `must not`, `should`, `should not`, and `may` consistently.
- **Security first**: security requirements must be expressed before convenience features.
- **Reviewability**: requirements must be understandable in plain text and suitable for code review.
- **Testability**: every mandatory behavior should have an associated validation method.
- **AI-agent compatibility**: documents should be structured so AI coding agents can convert them into tasks, tests, and implementation plans.
- **No hidden defaults**: any default that affects security, tenancy, identity, persistence, or deployment must be explicit.

## 6. Phase 0 artifact map

The Phase 0 specification package should use this documentation structure:

| Artifact | Proposed path | Purpose | Required before Phase 1 |
| --- | --- | --- | --- |
| Main framework specification | `docs/specification.md` | Project-wide requirements and resolved baseline decisions | Yes |
| Phase 0 specification package | `docs/phase-0-specification.md` | Phase 0 scope, deliverables, process, and readiness checklist | Yes |
| Architecture specification | `docs/architecture.md` | Component contracts, data flow, extension points, and generated artifact boundaries | Yes |
| Security specification | `docs/security.md` | Secure defaults, identity, authorization, tenancy, SQL safety, audit, secrets, and supply-chain rules | Yes |
| Threat model | `docs/threat-model.md` | Assets, trust boundaries, threats, mitigations, and validation strategy | Yes |
| Resource model specification | `docs/resource-model.md` | Resource authoring format, field model, tenancy declarations, policies, and generated artifacts | Yes |
| CLI specification | `docs/cli.md` | Commands, inputs, outputs, plans, errors, and exit codes | Yes |
| Prototype acceptance plan | `docs/prototype-acceptance.md` | Phase 1 acceptance tests and demo scenario | Yes |
| Decision log | `docs/decisions.md` | Resolved and deferred decisions with rationale | Yes |

This repository may introduce these files incrementally. Until all files exist, this Phase 0 document acts as the coordination document and readiness checklist.

## 7. Product specification requirements

The product specification must define:

- Target users: developers, platform teams, security reviewers, AI coding agents, and operators building operational business systems.
- Primary use cases: internal operational APIs, CRUD workflows, approval systems, data stewardship tools, tenant-aware back-office services, and AI-agent-operated workflows.
- Non-goals: low-code visual builders, GUI-first administration, password-login starter kits, proprietary hosted control planes, and opaque plugin behavior.
- Adoption model: teams can use Daicho for one service or workflow without migrating their full platform.
- First prototype boundary: one tenant-scoped resource, local PostgreSQL, local HTTP API, deny-by-default policy, audit events, structured logs, and Docker Compose deployment.

The product specification must also distinguish between:

- **Core framework behavior**, which must be implemented by Daicho.
- **Generated application behavior**, which Daicho produces in user projects.
- **Optional adapters**, which may be added without changing secure defaults.
- **External infrastructure**, such as upstream identity proxies and managed PostgreSQL providers.

## 8. Architecture specification requirements

The architecture specification must define at least these components and contracts.

### 8.1 CLI

The CLI is the only required developer interface. It must be scriptable, deterministic, and usable by humans and AI agents.

The architecture specification must describe how the CLI:

- Reads project configuration.
- Validates resource definitions.
- Produces plans before mutations.
- Generates source code and SQL files.
- Runs local checks.
- Exports OpenAPI and JSON Schema.
- Creates deployment templates.
- Reports machine-readable errors.

### 8.2 Schema compiler

The schema compiler converts resource definitions into an internal representation. It must:

- Reject nondeterministic definitions.
- Preserve stable names for generated artifacts.
- Track source locations for diagnostics.
- Normalize field definitions, relationships, policy bindings, tenant settings, audit settings, and API exposure.
- Emit an intermediate representation that can be inspected in tests.

### 8.3 Code generator

The generator must produce reviewable code. It must not hide security-critical behavior behind opaque runtime magic.

Generated code must show:

- Tenant constraints.
- Policy checks.
- Validation calls.
- Transaction boundaries.
- Audit event writes.
- Safe SQL construction.

### 8.4 Runtime

The runtime must be minimal and explicit. The architecture specification must define request flow from HTTP ingress to validation, identity extraction, tenant context resolution, authorization, database access, audit logging, and response serialization.

### 8.5 PostgreSQL layer

The PostgreSQL layer must be defined as a security boundary. It must prevent raw string interpolation for untrusted values and must enforce tenant-aware query construction for tenant-scoped resources.

### 8.6 Adapter boundary

Adapters must be optional and explicit. The architecture specification must define how identity providers, object storage, queues, email providers, and deployment targets integrate without becoming mandatory hidden infrastructure.

## 9. Security specification requirements

The security specification must be written before security-sensitive code is implemented. It must include the following mandatory rules.

### 9.1 Secure defaults

Default generated projects must:

- Disable password login.
- Require an explicit identity mode.
- Deny all resource operations unless policies allow them.
- Require tenancy declarations for every resource.
- Use safe SQL construction.
- Avoid default public administrative screens.
- Avoid logging secrets, tokens, session values, or full request bodies by default.
- Include local security checks in generated project scripts.

### 9.2 Authentication

The first identity mode should be upstream trusted identity / no-login mode. It may trust identity headers only when the deployment explicitly configures a trusted boundary. The specification must document how spoofed headers are prevented and how local development simulates identity safely.

OIDC, SAML, passkeys, service credentials, and password authentication must be treated as separate explicit adapters. Password authentication must remain opt-in if it is ever implemented.

### 9.3 Authorization

Authorization must be deny-by-default. Policies must be evaluated before data access unless the operation requires an existing record; in that case, the existing-record read must itself be tenant constrained and minimal.

The policy specification must define:

- Principal shape.
- Tenant context shape.
- Resource and operation names.
- Existing and proposed record values.
- Request metadata available to policies.
- Deterministic test harness behavior.
- Audit output for allow and deny decisions.

### 9.4 Tenancy

Tenant isolation must be enforced by construction. The specification must define behavior for `global`, `tenant_scoped`, `tenant_partitioned`, and `system` resources.

For `tenant_scoped` resources, generated queries must include tenant constraints in every read, update, delete, relationship traversal, and uniqueness check where applicable.

### 9.5 SQL safety

The SQL safety specification must define:

- Parameter binding requirements.
- Identifier escaping rules.
- Forbidden raw SQL APIs in generated code.
- Review requirements for custom SQL escape hatches.
- Query-builder invariants.
- Tests for injection attempts and missing tenant constraints.

### 9.6 Audit and logs

Audit events must be append-only from the generated application perspective. The specification must define required fields, redaction rules, correlation IDs, retention assumptions, and failure behavior when audit persistence fails.

### 9.7 Supply chain

The supply-chain section must define dependency selection rules, lockfile requirements, vulnerability scanning expectations, SBOM path, release signing path, template integrity checks, and minimum CI checks.

## 10. Threat model requirements

The threat model must cover at least:

- Assets: tenant data, credentials, audit records, generated source, migration files, deployment manifests, and policy definitions.
- Actors: legitimate users, tenant administrators, operators, external attackers, malicious dependencies, compromised AI agents, and misconfigured identity providers.
- Trust boundaries: browser/client, upstream identity proxy, generated application, PostgreSQL, deployment platform, CI/CD, package registry, and optional adapters.
- Threats: cross-tenant access, SQL injection, authorization bypass, template tampering, dependency compromise, secret leakage, audit tampering, unsafe migrations, and AI-agent mistakes.
- Mitigations: design constraints, generated tests, static checks, runtime guards, review steps, and deployment guidance.
- Residual risks: explicit risks accepted for the first prototype and reasons they are acceptable.

Threats should be prioritized by likelihood and impact. Each high-priority threat must map to one or more required controls and one or more validation methods.

## 11. Resource model specification requirements

The resource model specification must define the first supported authoring format and the internal representation that generators consume.

At minimum, a resource must define:

- Stable name.
- Tenancy mode.
- Field list.
- Primary key.
- Tenant key when tenant-scoped.
- Validation constraints.
- Indexes.
- Relationships.
- API exposure.
- Policy bindings.
- Audit behavior.

The specification must define how resources are versioned, how renames are represented, how migrations are generated, and which changes require manual review.

### 11.1 Example resource sketch

The final resource specification should include an example similar to this non-normative sketch:

```ts
export const customer = resource({
  name: "customer",
  tenancy: "tenant_scoped",
  table: "customers",
  fields: {
    id: uuid().primaryKey(),
    tenantId: uuid().tenantKey(),
    name: text().min(1).max(200).required(),
    status: enumText(["active", "archived"]).default("active"),
    createdAt: timestamp().generated(),
    updatedAt: timestamp().generated(),
  },
  api: {
    operations: ["create", "read", "list", "update", "delete"],
  },
  policies: {
    create: "customer.create",
    read: "customer.read",
    list: "customer.list",
    update: "customer.update",
    delete: "customer.delete",
  },
  audit: {
    events: ["create", "update", "delete", "policy_denied"],
  },
});
```

This sketch is illustrative only. Phase 0 must decide the actual syntax before Phase 1 implementation begins.

## 12. CLI specification requirements

The CLI specification must define command behavior in enough detail to support test-first implementation.

Required command groups:

- `daicho init`: create a project from a template.
- `daicho validate`: validate configuration, resources, policies, and deployment manifests.
- `daicho plan`: emit a machine-readable plan for generation, migration, or deployment actions.
- `daicho generate`: generate source, schemas, OpenAPI, and SQL artifacts.
- `daicho migrate`: create and apply PostgreSQL migrations.
- `daicho policy test`: run policy tests without a browser or GUI.
- `daicho test`: run generated project checks.
- `daicho deploy prepare`: create deployment bundles or templates without applying infrastructure.
- `daicho doctor`: inspect environment prerequisites and configuration risks.

For each command, the specification must define:

- Required and optional flags.
- Input files read.
- Output files written.
- Whether the command mutates state.
- Dry-run behavior.
- Plan output schema.
- JSON output mode.
- Exit codes.
- Error categories.
- Logging and redaction behavior.

## 13. Plan format requirements

Commands that propose changes must emit plans before applying changes. The plan format must be stable enough for review tools and AI agents.

A plan must include:

- Plan schema version.
- Daicho version.
- Project root.
- Command and arguments.
- Timestamp.
- Inputs and content hashes.
- Proposed file changes.
- Proposed database changes.
- Proposed deployment changes.
- Security-sensitive changes.
- Warnings.
- Required approvals, if any.

Plans must not include secrets. Any redacted value must be marked as redacted rather than omitted when its presence affects review.

## 14. Prototype acceptance plan requirements

The Phase 1 prototype acceptance plan must define a single end-to-end scenario:

1. Initialize a new project.
2. Define one tenant-scoped resource.
3. Validate the resource.
4. Generate migration SQL and TypeScript runtime code.
5. Start local PostgreSQL and the generated application.
6. Make create, read, list, update, and delete requests with a simulated upstream identity.
7. Prove that cross-tenant reads and writes fail.
8. Prove that missing policies deny access.
9. Prove that audit events are written.
10. Export OpenAPI and JSON Schema.
11. Run all checks from the command line.
12. Produce a Docker Compose deployment.

The plan must identify exact commands, expected files, expected HTTP responses, expected audit records, and expected failure cases.

## 15. Decision log requirements

The decision log must record:

- Decision identifier.
- Title.
- Status: proposed, accepted, superseded, or rejected.
- Context.
- Decision.
- Consequences.
- Alternatives considered.
- Date accepted.
- Reviewers or owners.

At minimum, Phase 0 must record decisions for:

- First resource definition format.
- First identity mode.
- SQL generation strategy.
- PostgreSQL row-level security posture.
- Minimum deployment target.
- Policy authoring model.
- Generated code ownership model.
- Dependency policy.

## 16. Documentation quality bar

Before Phase 0 is accepted, documents must satisfy this quality bar:

- Requirements are written in English.
- Normative statements are unambiguous.
- Security-sensitive defaults are explicit.
- Each major requirement has a validation method.
- Open questions are listed separately from resolved decisions.
- Examples are labeled as non-normative when they are not binding.
- File names and command names are stable enough for Phase 1 planning.
- The first prototype can be implemented from the documents without relying on private context.

## 17. Phase 0 review workflow

Phase 0 changes should follow this workflow:

1. Open a documentation change with a clear summary and affected areas.
2. Identify whether the change modifies security, tenancy, identity, persistence, deployment, or generated-code behavior.
3. Add or update validation criteria for any new mandatory behavior.
4. Update the decision log when a design decision changes.
5. Request review from at least one maintainer and one security-minded reviewer for security-sensitive changes.
6. Merge only after open questions are either resolved or explicitly deferred.

## 18. Phase 0 readiness checklist

Phase 0 is ready to close when all checklist items are complete:

- [ ] Main framework specification reviewed.
- [ ] Architecture specification created and reviewed.
- [ ] Security specification created and reviewed.
- [ ] Threat model created and reviewed.
- [ ] Resource model specification created and reviewed.
- [ ] CLI specification created and reviewed.
- [ ] Prototype acceptance plan created and reviewed.
- [ ] Decision log created and reviewed.
- [ ] Open questions assigned owners or deferred with rationale.
- [ ] Phase 1 implementation tasks created from accepted specifications.
- [ ] Security-critical acceptance tests identified before implementation begins.

## 19. Phase 1 entry criteria

Phase 1 may begin only after Phase 0 provides enough detail to implement and test the first prototype. Required entry criteria are:

- The first resource authoring format is accepted.
- The first identity mode is accepted.
- CLI command names and output expectations are accepted.
- Tenant isolation invariants are accepted.
- SQL safety invariants are accepted.
- Prototype acceptance scenario is accepted.
- Required generated artifacts are listed.
- Deferred decisions are not blockers for the first prototype.

If any entry criterion is unresolved, Phase 1 work may still explore prototypes, but those prototypes must be treated as disposable spikes rather than framework implementation.

## 20. Open questions for Phase 0

Phase 0 must resolve or explicitly defer these questions:

1. Should the first resource format be TypeScript-only, YAML/JSON-only, or dual-format?
2. Should generated code be fully checked into user repositories, partially generated, or produced on demand?
3. What is the minimum safe custom SQL escape hatch?
4. What policy language should be used first: TypeScript functions, declarative expressions, or both?
5. What is the exact local development identity simulation model?
6. Which Node.js LTS and PostgreSQL version ranges should the first prototype target?
7. Which package manager should generated projects use by default?
8. Which SBOM tool and vulnerability scanner should be recommended first?
9. How should migration rename detection and destructive-change approval work?
10. What level of row-level security support belongs in Phase 1 versus later phases?
