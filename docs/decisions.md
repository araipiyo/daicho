# Daicho Decision Log

Status: Draft 0.4
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

### D0101: First resource authoring format

Status: Accepted
Context: Phase 1 needs a format that AI agents can read and write reliably, humans can review as text, the CLI can validate without executing application code, and deployment tooling can map to PostgreSQL without ambiguity. SQL DDL is easy for AI agents to understand and easy to apply, but it is a weak source for API validation, tenancy metadata, policy bindings, model typing, audit behavior, and destructive-change classification. A TypeScript DSL is ergonomic for application developers, but arbitrary TypeScript execution weakens determinism and security.
Decision: The first canonical resource format is strict JSON files validated by JSON Schema, stored under `resources/*.resource.json`. JSON is the source of truth for resources. The CLI may generate SQL DDL migrations, TypeScript model types, OpenAPI, and JSON Schema from this resource JSON, but generated artifacts are not authoritative. JSON must use closed schemas, explicit enum values, stable names, explicit database mappings, and no implicit defaults except those named in the schema. A small TypeScript builder DSL may be explored later only if it emits the same canonical JSON and is validated by the same checker. Hand-written SQL DDL may appear only as migration output or as reviewed manual migrations, not as the primary resource format.
Consequence: Resource authoring is deterministic, AI-friendly, reviewable, and deployable through generated SQL while keeping policy, validation, tenancy, API, and audit metadata in one complete artifact. The schema checker becomes the security gate before SQL generation or deployment.
Alternatives considered: SQL DDL as source of truth was rejected because too much security and application metadata would live elsewhere. Arbitrary TypeScript resource modules were rejected for Phase 1 because static validation without executing code is harder. YAML was rejected because duplicate keys, anchors, and implicit typing increase ambiguity. A custom DSL was deferred because JSON plus JSON Schema is simpler and more toolable for Phase 1.
Date accepted: 2026-05-11.
Owners or reviewers: Maintainers.

### D0102: Generated or starter code ownership and overwrite model

Status: Accepted
Context: Phase 1 starts from a normal `git clone` of a maintained starter template. AI agents need freedom to edit the project, but security-critical runtime boundaries and generated files must stay understandable.
Decision: The starter repository contains runnable starter code that AI agents may modify like normal application code. The starter must include AI-facing Markdown instructions that describe project layout, ownership boundaries, safe commands, security invariants, and which files are generated. Generated artifacts must include a header naming the generator, source resource or policy file, generation time or deterministic build identity, and overwrite policy. Generated files may be overwritten only by `daicho export`, migration generation, or an explicitly documented command; application-owned files are never silently overwritten.
Consequence: A cloned project is immediately useful and AI-editable, while reviewers can distinguish application code, framework-owned runtime code, generated exports, and manual migrations.
Alternatives considered: A generator-only workflow was rejected by D0007. Locking starter code against AI edits was rejected because it harms the primary workflow. Silent regeneration was rejected because it hides security-relevant diffs.
Date accepted: 2026-05-11.
Owners or reviewers: Maintainers.

### D0103: First policy authoring model

Status: Accepted
Context: Authorization is deny-by-default, but the first policy language should remain small because the maintainer has not yet committed to a full external policy engine. Existing models include Cedar, OPA/Rego, OpenFGA/Zanzibar-style tuples, and application code checks. Cedar has a JSON policy format and a schema concept; OPA/Rego is powerful for general policy-as-code; OpenFGA is strong for relationship authorization.
Decision: The first policy authoring model is a Daicho-owned strict JSON policy binding format under `policies/*.policy.json`, with an intentionally small expression vocabulary for Phase 1. Policies bind resource operations to named rules that evaluate principal attributes, tenant context, resource attributes, proposed changes, relationship predicates exposed by the runtime, and request metadata. The runtime must deny missing bindings and unsupported expressions. The JSON shape should be designed so it can later compile to Cedar, OPA/Rego, or OpenFGA-backed checks if Daicho adopts one of those engines.
Consequence: Phase 1 can ship a minimal, auditable, no-execution policy format while preserving an escape route toward mature policy systems.
Alternatives considered: Cedar was the strongest candidate for future adoption because it is purpose-built for application authorization and supports JSON policies and schemas. OPA/Rego was deferred because it is broader than Daicho needs for first resource CRUD authorization. OpenFGA was deferred because relationship tuples are useful later but do not by themselves cover tenant, attribute, and proposed-change checks. TypeScript policy functions were rejected as the first authoring model because arbitrary code is harder to statically inspect and safely explain.
Date accepted: 2026-05-11.
Owners or reviewers: Maintainers.

### D0104: Minimum safe custom SQL escape hatch

Status: Accepted
Context: Phase 1 needs a minimal way to express custom SQL when generated helpers are insufficient. The maintainer requested a string-operation engine if the scope is minimum, while also asking for TypeScript libraries comparable to Ruby Sequel.
Decision: Custom SQL is disabled by default and must use an approved tagged-template helper, `sql`, that parameterizes values and only permits identifiers through generated or allow-listed identifier tokens. The helper may concatenate only already-tokenized SQL fragments, never raw untrusted strings. Each custom SQL use must declare resource scope, tenant-safety rationale, expected operation, test coverage, and review status. Kysely is the preferred TypeScript library to evaluate for richer query-building because it is a thin, type-safe SQL query builder with PostgreSQL support, raw SQL primitives, and optional migration primitives. Drizzle may be evaluated for schema-to-SQL generation, but it must not replace the canonical resource JSON in Phase 1.
Consequence: Daicho gets a small, searchable escape hatch with clear review gates now, and a migration path toward a mature TypeScript query builder later.
Alternatives considered: Free-form SQL strings were rejected. node-postgres alone was considered too low-level for safe composition. Knex was not selected as first choice because Kysely better matches modern TypeScript typing goals. A full ORM was deferred because the runtime needs predictable SQL boundaries more than object persistence magic.
Date accepted: 2026-05-11.
Owners or reviewers: Maintainers.

### D0105: First package manager

Status: Accepted
Context: The package manager affects lockfile determinism, install behavior, CI reproducibility, and supply-chain risk. Security is the priority.
Decision: Use `pnpm` as the first package manager, pinned through Corepack with an exact `packageManager` value in `package.json`. CI and local docs must use `pnpm install --frozen-lockfile`. Dependency installation should disable lifecycle scripts by default for CI checks where practical, and any required build scripts must be allow-listed and documented.
Consequence: The starter gets a deterministic lockfile, strict dependency layout, and a single package-manager path for AI agents and CI.
Alternatives considered: npm was considered because it is bundled with Node.js and supports audit/signature workflows, but pnpm's strict dependency model and workspace ergonomics are preferable for the starter. Yarn and Bun were deferred to avoid adding another runtime or package-manager surface.
Date accepted: 2026-05-11.
Owners or reviewers: Maintainers.

### D0106: Node.js and PostgreSQL version ranges

Status: Accepted
Context: Runtime versions must balance security support, managed-service availability, and upgrade pressure. As of 2026-05-11, Node.js 24 is Active LTS, Node.js 22 is Maintenance LTS, and PostgreSQL 18, 17, 16, and 15 remain supported upstream.
Decision: Phase 1 supports Node.js `>=24 <25` for production and CI. Node.js 22 may be used only as a temporary local compatibility lane until the starter is fully on Node.js 24, and it must not be the default. Phase 1 supports PostgreSQL `>=17 <19`, with PostgreSQL 18 preferred and PostgreSQL 17 accepted for managed providers or conservative deployments. PostgreSQL 16 and older are not default targets for new Phase 1 projects. Patch and minor releases must track the latest security release in the selected major line.
Consequence: Daicho starts on currently supported LTS/runtime lines with enough PostgreSQL runway while avoiding nearly end-of-life database majors.
Alternatives considered: Supporting PostgreSQL 15 or 16 would increase managed-provider compatibility but shortens security runway and broadens testing. Supporting Node.js Current releases was rejected for production because LTS stability matters more.
Date accepted: 2026-05-11.
Owners or reviewers: Maintainers.

### D0107: SBOM and vulnerability scanning tools

Status: Accepted
Context: Daicho needs machine-readable dependency inventory and vulnerability gates for AI-assisted development.
Decision: Generate CycloneDX JSON SBOMs with `@cyclonedx/cdxgen`. Scan dependencies with OSV-Scanner against committed lockfiles and generated SBOMs. Also run the package-manager native audit path where available, including npm registry signature/provenance verification for npm-hosted packages when the toolchain supports it. CI must fail on known critical vulnerabilities unless a time-limited, reviewed exception is committed.
Consequence: The starter has a vendor-neutral SBOM format, an open vulnerability scanner, and a second ecosystem-native signal for npm package integrity.
Alternatives considered: Syft and Grype are viable alternatives, especially for container images, but cdxgen plus OSV-Scanner is the first source dependency path. Commercial scanners can be added by adopters but are not required for Phase 1.
Date accepted: 2026-05-11.
Owners or reviewers: Maintainers.

### D0108: Destructive migration approval workflow

Status: Accepted
Context: Destructive changes must not be applied silently. The migration tool also influences the resource authoring model. Drizzle can generate SQL from TypeScript schemas, node-pg-migrate provides PostgreSQL migration files, and Kysely provides optional migration primitives. No evaluated library replaces the need for Daicho-specific destructive-change classification tied to canonical resources.
Decision: Phase 1 uses Daicho-generated SQL migration files from canonical resource JSON, applied by a small Daicho migration runner that records applied migrations and checksums in PostgreSQL. Kysely migration primitives may be used internally if they reduce implementation risk, but the human-reviewed artifact remains SQL. Destructive migrations require a plan that classifies the operation, shows affected tables/columns/indexes without secrets, names backup/rollback expectations, and refuses apply unless a human creates an approval file or passes an explicit approval token matching the migration checksum. AI agents may prepare plans but must not auto-approve destructive migrations.
Consequence: D0101 and D0108 share one source of truth: resource JSON produces reviewable SQL, and Daicho owns destructive analysis instead of delegating it blindly to a general migration library.
Alternatives considered: Drizzle migrations were considered but would make TypeScript schema a competing source of truth. node-pg-migrate was considered for imperative migrations but does not itself solve Daicho resource metadata or destructive approval. Direct hand-written SQL remains allowed for manual migrations but must go through the same checksum, plan, review, and approval flow.
Date accepted: 2026-05-11.
Owners or reviewers: Maintainers.

## 4. Deferred Phase 0 decisions

### D0109: Row-level security roadmap

Status: Deferred
Decision: Not needed for Phase 1. Tenant safety is enforced in the runtime and approved query helpers first. PostgreSQL row-level security may be reconsidered after the first prototype proves the resource and policy model.
Date deferred: 2026-05-11.

### D0110: Provider-specific protected-ingress adapters and proof formats

Status: Deferred
Decision: Not needed for Phase 1 beyond the generic protected-ingress model already accepted in D0008. Provider-specific adapters and proof formats are deferred until deployment targets are selected.
Date deferred: 2026-05-11.

### D0111: Post-Phase 1 runtime distribution mechanism

Status: Deferred
Decision: Not needed before Phase 1. D0009 remains the controlling runtime-linkage decision: the starter may colocate runtime code now, while the preferred future is a separately versioned, provenance-checked runtime dependency.
Date deferred: 2026-05-11.
