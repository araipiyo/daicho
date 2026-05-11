# Daicho CLI Specification

Status: Draft 0.1  
Phase: 0

## 1. Purpose

This document defines the command-line behavior required for the first Daicho prototype.

## 2. Global CLI requirements

The CLI must:

- Run without a GUI.
- Support human-readable output by default.
- Support JSON output for automation.
- Return stable exit codes.
- Redact secrets.
- Emit plans before mutating infrastructure or databases.
- Be deterministic for the same inputs.

## 3. Exit codes

The first prototype should reserve these exit codes:

- `0`: success.
- `1`: general failure.
- `2`: validation failure.
- `3`: configuration error.
- `4`: plan contains unapproved destructive changes.
- `5`: environment prerequisite failure.
- `6`: security invariant failure.

## 4. Required commands

### 4.1 `daicho init`

Creates a new project from a template. It must not enable password login by default.

### 4.2 `daicho validate`

Validates project configuration, resources, policies, and deployment manifests. It must fail on missing tenancy declarations, missing required policy bindings, and unsafe configuration.

### 4.3 `daicho plan`

Emits a machine-readable plan for generation, migration, or deployment work. Plans must not include secrets.

### 4.4 `daicho generate`

Generates TypeScript, SQL, OpenAPI, JSON Schema, and supporting files. It must identify generated files and preserve reviewability.

### 4.5 `daicho migrate`

Creates or applies PostgreSQL migrations. Destructive operations must require an explicit approval mechanism.

### 4.6 `daicho policy test`

Runs policy tests without a browser or GUI. It must support negative tests for deny-by-default behavior.

### 4.7 `daicho test`

Runs generated project checks from the command line.

### 4.8 `daicho deploy prepare`

Creates deployment bundles or templates without applying infrastructure.

### 4.9 `daicho doctor`

Checks local prerequisites, configuration risks, and common environment problems.

## 5. Plan schema requirements

A plan must include:

- Plan schema version.
- Daicho version.
- Project root.
- Command and arguments.
- Timestamp.
- Input file hashes.
- Proposed file changes.
- Proposed database changes.
- Proposed deployment changes.
- Security-sensitive warnings.
- Required approvals.

## 6. Error output requirements

Errors must include:

- Stable error code.
- Short message.
- Detailed message.
- Source location when applicable.
- Suggested remediation when safe.

Errors must not include secrets.
