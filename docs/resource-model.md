# Daicho Resource Model Specification

Status: Draft 0.2
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

## 9. Non-normative example

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

The exact authoring format remains a Phase 0 decision.
