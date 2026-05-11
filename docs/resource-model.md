# Daicho Resource Model Specification

Status: Draft 0.1  
Phase: 0

## 1. Purpose

This document defines the resource model that Daicho will use to generate validation, database, API, policy, and audit artifacts.

## 2. Resource requirements

Every resource must define:

- Stable resource name.
- Tenancy mode.
- Database table mapping.
- Primary key.
- Field definitions.
- Validation constraints.
- Indexes.
- Relationships.
- API exposure.
- Policy bindings.
- Audit behavior.

## 3. Tenancy modes

A resource must use one of:

- `global`.
- `tenant_scoped`.
- `tenant_partitioned`.
- `system`.

A `tenant_scoped` resource must define a tenant key. Generated queries must include tenant constraints for all operations that access tenant data.

## 4. Field model

Fields must have stable names and explicit types. The first prototype should support at least:

- UUID.
- Text.
- Integer.
- Boolean.
- Timestamp.
- Enum-like text.
- JSON value, if a safe validation strategy is specified.

Fields may declare:

- Required or optional status.
- Defaults.
- Generated values.
- Minimum and maximum length.
- Numeric bounds.
- Uniqueness.
- Index participation.
- Redaction behavior for audit and logs.

## 5. API exposure

Resources must explicitly list exposed operations. Supported first-prototype operations are:

- `create`.
- `read`.
- `list`.
- `update`.
- `delete`.

Operations not listed must not be generated.

## 6. Policy bindings

Every exposed operation must bind to a policy. Missing policies must fail validation or produce deny-by-default behavior with a clear diagnostic.

## 7. Audit behavior

Resources must define which events are audited. Security-relevant events should include create, update, delete, policy denial, and tenant-boundary failures.

## 8. Migration behavior

Resource changes must be classified as:

- Safe additive changes.
- Potentially destructive changes.
- Rename-like changes requiring explicit mapping.
- Manual migration changes.

Destructive changes must require an explicit plan warning and manual approval before application.

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

This syntax is illustrative until the Phase 0 decision log accepts the first resource authoring format.
