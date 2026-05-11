# Daicho CLI Specification

Status: Draft 0.3
Phase: 0

## 1. Purpose

This document defines the Phase 1 command-line surface for developers, AI coding agents, CI, and limited operator workflows. It is not a product-user interface.

## 2. Global requirements

The CLI must:

- Run without a GUI.
- Be safe for AI coding agents such as Codex or Claude Code to invoke in development and CI workflows.
- Avoid any requirement that product users install or run the CLI.
- Default to human-readable output.
- Support JSON output for automation.
- Use stable exit codes and diagnostic codes.
- Be deterministic for the same inputs.
- Redact secrets.
- Emit plans before data or deployment changes.

## 3. Exit codes

Reserved exit codes:

- `0`: success.
- `1`: general failure.
- `2`: validation failure.
- `3`: configuration error.
- `4`: unapproved destructive plan.
- `5`: environment prerequisite failure.
- `6`: security invariant failure.

## 4. Required commands

### `daicho check`

Validates configuration, resources, policies, tenancy declarations, deployment files, runtime linkage, protected-ingress assumptions, and dependency risk. It must fail on missing tenancy, missing policy bindings, unsafe identity configuration, unverified upstream trusted identity, direct-origin exposure without an accepted exception, and unsafe default security settings.

### `daicho test`

Runs the project test suite and Daicho security checks. It must support command-line policy tests and negative tests for deny-by-default behavior.

### `daicho export`

Writes OpenAPI and JSON Schema artifacts. Exports must be deterministic, reviewable, and free of secrets.

### `daicho deploy prepare`

Prepares deployment files, redacted plans, and human-readable instructions. It must not apply infrastructure. The instructions should prefer protected ingress through Cloudflare Access, Tailscale, identity-aware proxies, API gateways, private service meshes, or equivalent products, and should warn against direct public origin exposure.

### `daicho doctor`

Reports local prerequisites, configuration risks, runtime linkage state, protected-ingress configuration, and common environment problems.

## 5. Plan requirements

A plan must include:

- Schema version.
- Daicho version.
- Runtime linkage version or commit.
- Project root.
- Command and arguments.
- Timestamp.
- Input file hashes.
- Proposed file changes.
- Proposed database changes.
- Proposed deployment changes.
- Protected-ingress mode and origin-exposure warnings.
- Security warnings.
- Required approvals.

A plan must not include secrets.

## 6. Diagnostic requirements

Diagnostics must include:

- Stable code.
- Severity.
- Short message.
- Detailed message.
- Source location when applicable.
- Safe remediation guidance.

Diagnostics must not include secrets.

## 7. Explicit non-goals for Phase 1

Phase 1 does not require:

- `daicho init`.
- General project scaffolding.
- Full application code generation.
- Automatic cloud deployment.
- Product-user workflows that require the Daicho CLI.
