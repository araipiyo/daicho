# Daicho Resource Model Specification

Status: Draft 0.3
Phase: 0

## 1. Purpose

This document defines resources: the source model Daicho uses for validation, database mapping, APIs, policies, and audit.

## 2. Required resource fields

Every resource must declare:

- Stable name.
- Tenancy mode.
- Database table mapping.
- Primary key.
- Fields and validation.
- Indexes.
- Relationships.
- Exposed API operations.
- Policy bindings.
- Audit behavior.

## 3. Tenancy modes

A resource must use exactly one mode:

- `global`: shared data protected by policy.
- `tenant_scoped`: each row belongs to one tenant.
- `tenant_partitioned`: data is isolated by schema or database.
- `system`: framework or operational data.

`tenant_scoped` resources must define a tenant key. All reads, lists, updates, deletes, relationship traversal, uniqueness checks, and existing-record lookups must include tenant constraints.

## 4. Field model

Fields must have stable names and explicit types. The first prototype should support:

- UUID.
- Text.
- Integer.
- Boolean.
- Timestamp.
- Enum-like text.
- JSON value only with explicit validation.

Fields may declare required status, defaults, generated values, length bounds, numeric bounds, uniqueness, indexes, and redaction behavior.

## 5. API exposure

Resources expose only explicitly listed operations:

- `create`.
- `read`.
- `list`.
- `update`.
- `delete`.

Unlisted operations must not be available.

## 6. Policy bindings

Every exposed operation must bind to a policy. Missing bindings must fail validation or deny access with a clear diagnostic.

## 7. Audit behavior

Resources must declare audited events. Security-relevant events should include create, update, delete, policy denial, and tenant-boundary failure.

## 8. Migration behavior

Resource changes must be classified as:

- Safe additive.
- Potentially destructive.
- Rename-like and requiring explicit mapping.
- Manual migration.

Destructive changes require plan warnings and explicit approval before application.


## 9. First authoring format

The first canonical resource authoring format is strict JSON validated by JSON Schema. Resource files live under `resources/*.resource.json` and are the source of truth for validation, database mapping, API generation, policy binding, audit settings, and migration planning.

A resource JSON document must be complete and unambiguous:

- No implicit tenancy, table, primary key, API operation, policy binding, audit event, or destructive-change behavior.
- Only schema-defined keys are allowed. Unknown keys fail validation.
- Enums use explicit string values.
- Defaults must be declared in the resource document and must identify whether they are application defaults, database defaults, or generated values.
- Database identifiers must be explicit and checked against Daicho identifier rules.
- JSON fields require explicit validation schemas and redaction settings.

The CLI may generate SQL DDL migrations, TypeScript model types, OpenAPI, JSON Schema, and policy test scaffolds from the resource JSON. Generated artifacts are reviewable outputs, not the canonical source. A future TypeScript DSL is allowed only if it emits the same canonical JSON without executing arbitrary application code during validation. YAML is not used as the first format because duplicate keys, anchors, and implicit typing introduce avoidable ambiguity.

## 10. Non-normative JSON example

```json
{
  "name": "customer",
  "tenancy": {
    "mode": "tenant_scoped",
    "tenantKey": "tenantId"
  },
  "table": "customers",
  "primaryKey": ["id"],
  "fields": {
    "id": { "type": "uuid", "primaryKey": true, "generated": "uuid" },
    "tenantId": { "type": "uuid", "tenantKey": true, "required": true },
    "name": { "type": "text", "required": true, "minLength": 1, "maxLength": 200 },
    "status": { "type": "text", "enum": ["active", "archived"], "default": { "application": "active" } },
    "createdAt": { "type": "timestamp", "generated": "now" },
    "updatedAt": { "type": "timestamp", "generated": "now" }
  },
  "indexes": [
    { "name": "customers_tenant_status_idx", "columns": ["tenantId", "status"] }
  ],
  "api": { "operations": ["create", "read", "list", "update", "delete"] },
  "policies": {
    "create": "customer.create",
    "read": "customer.read",
    "list": "customer.list",
    "update": "customer.update",
    "delete": "customer.delete"
  },
  "audit": { "events": ["create", "update", "delete", "policy_denied"] },
  "migrations": { "destructiveChanges": "explicit_approval" }
}
```

## 11. Non-normative TypeScript-style example

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
  api: { operations: ["create", "read", "list", "update", "delete"] },
  policies: {
    create: "customer.create",
    read: "customer.read",
    list: "customer.list",
    update: "customer.update",
    delete: "customer.delete",
  },
  audit: { events: ["create", "update", "delete", "policy_denied"] },
});
```

This TypeScript-style example is illustrative only. The canonical Phase 1 source is the strict JSON format above.
