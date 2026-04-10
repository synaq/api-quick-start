# Product Migration and Domain Move Guide

---

## Migrating a domain to a new product

To move a domain from one SYNAQ product to another, you create a new package representing the target product, then link the existing domain to it. The API evaluates the link request, automatically unlinks the old package, and triggers any necessary decommissioning.

### Prerequisites

- The new package must be created under the **same company** as the domain
- The `expect_migration` flag must be raised on the new package
- If the new product lacks a mailbox service, all active mailboxes must be deleted first

### Check eligibility first (optional but recommended)

Before linking, check whether the migration can proceed:

```
OPTIONS /api/v1/packages/{new-package-guid}/domains/{domain-guid}.json
```

| Response         | Meaning |
|------------------|---------|
| `204 No Content` | Migration can proceed |
| `200 OK`         | Migration can proceed; response includes mailbox edition mapping report |
| `409 Conflict`   | Migration blocked — inspect response body for reason |

**Edition mapping report (for migrations between products with mailbox editions):**

```json
{
  "mailbox_edition_map": [
    {
      "current_edition": "{current-edition-code}",
      "new_edition": "{new-edition-code}",
      "count": 2
    }
  ]
}
```

---

### Example 1: Migrating between products (same service set)

**Step 1 — Create the new package with `expect_migration`:**

```
POST /api/v1/ous/{company-guid}/packages.json
```

```json
{
  "package": {
    "code": "{new-package-code}",
    "expect_migration": true
  }
}
```

```
201 Created
Location: /api/v1/packages/{new-package-guid}
```

**Step 2 — Link the domain to the new package:**

```
LINK /api/v1/packages/{new-package-guid}/domains/{domain-guid}.json
```

```
204 No Content
```

The domain switches from `active` to `inactive`. This is expected — it needs to be reconfigured and reprovisioned on the new services. Backend services are not removed; the API updates mailbox editions in the background.

**Step 3 — Configure service fields:**

```
PATCH /api/v1/domains/{domain-guid}/servicefields.json
```

```json
{
  "fields": {
    "auth_method": "smtp",
    "username": "smtp-user",
    "password": "smtpPassw0rd!",
    "smtp_username": "smtp-user",
    "smtp_password": "smtpPassw0rd!",
    "admin_username": "admin@acme.com",
    "admin_password": "adminPassw0rd!",
    "admin_first_name": "Jane",
    "admin_last_name": "Doe",
    "service_provider": "other"
  }
}
```

**Step 4 — Provision the new package:**

```
POST /api/v1/packages/{new-package-guid}/actions.json
```

```json
{ "action": { "action": "Provision" } }
```

```
202 Accepted
Location: /api/v1/packages/{new-package-guid}/actions/{action-id}
```

Poll until `finished`. See [Async Actions](async-actions.md).

---

### Example 2: Migrating to a product that drops a service

When the new product excludes a service that the current product includes, the API automatically decommissions that service when the domain is linked to the new package. The LINK response returns an async `Prune` action instead of `204 No Content`.

**Step 1 — Create the new package:**

```
POST /api/v1/ous/{company-guid}/packages.json
```

```json
{
  "package": {
    "code": "{new-package-code}",
    "expect_migration": true
  }
}
```

**Step 2 — Link the domain:**

```
LINK /api/v1/packages/{new-package-guid}/domains/{domain-guid}.json
```

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{prune-action-id}
```

Poll the prune action until `finished`. Once complete, the dropped service is decommissioned and the domain is ready to be reconfigured.

> **Warning:** Services not included in the new product are decommissioned immediately when the old package is unlinked. Warn end users before performing migrations that drop services.

**Steps 3 and 4** — Configure service fields and provision the new package, as in Example 1.

---

## Moving a domain to a different company

Domains can be moved between any two companies in your scope. This is done by creating a new package under the destination company and linking the domain to it.

### Setup

When creating the package:
- Always raise `expect_domain_move`
- Raise `expect_migration` if also changing product (safe to always raise both)

### Example

**Step 1 — Create a new package under the destination company:**

```
POST /api/v1/ous/{destination-company-guid}/packages.json
```

```json
{
  "package": {
    "code": "{package-code}",
    "expect_domain_move": true,
    "expect_migration": true
  }
}
```

```
201 Created
Location: /api/v1/packages/{new-package-guid}
```

**Step 2 — Link the domain to the new package:**

The LINK response returns an async Move action:

```
LINK /api/v1/packages/{new-package-guid}/domains/{domain-guid}.json
```

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{move-action-id}
```

Poll the move action until `finished`. The domain is now under the new company and switches to `inactive` pending reconfiguration.

**Steps 3 and 4** — Configure service fields (possibly with new owner's credentials) and provision the package.
