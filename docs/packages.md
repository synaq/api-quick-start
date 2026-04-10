# Packages

A package is an instance of a SYNAQ product under a company. In the current API structure, each package is linked to exactly one domain.

## List available product combinations

Returns all valid product and edition combinations for a given company:

```
OPTIONS /api/v1/ous/{company-guid}/packages.json
```

Use this to dynamically generate a product selection form in your client UI.

---

## Get a package

```
GET /api/v1/packages/{package-guid}.json
```

Returns package details including current state and editions.

---

## Create a package

```
POST /api/v1/ous/{company-guid}/packages.json
```

**Request payload:**

```json
{
  "package": {
    "code": "{package-code}"
  }
}
```

**Response:**

```
201 Created
Location: /api/v1/packages/{package-guid}
```

### Migration flag

When creating a package for a domain migration (moving a domain from one product to another), raise the `expect_migration` flag:

```json
{
  "package": {
    "code": "{package-code}",
    "expect_migration": true
  }
}
```

### Domain move flag

When creating a package to receive a domain moving from a different company, raise the `expect_domain_move` flag. It is safe to also raise `expect_migration` at the same time to simplify implementation:

```json
{
  "package": {
    "code": "{package-code}",
    "expect_domain_move": true,
    "expect_migration": true
  }
}
```

---

## Provision a package

Used when provisioning a new package for an existing domain (e.g. after a migration). Follows the standard [async action](async-actions.md) pattern.

```
POST /api/v1/packages/{package-guid}/actions.json
```

**Request payload:**

```json
{
  "action": {
    "action": "Provision"
  }
}
```

**Response:**

```
202 Accepted
Location: /api/v1/packages/{package-guid}/actions/{action-id}
```

---

## Cancel a package

Cancels the package and all services associated with it. The domain linked to the package will be deprovisioned.

```
POST /api/v1/packages/{package-guid}/actions.json
```

**Request payload:**

```json
{
  "action": {
    "action": "Cancel"
  }
}
```

**Response:**

```
202 Accepted
Location: /api/v1/packages/{package-guid}/actions/{action-id}
```

Valid package actions: `Provision`, `Suspend`, `Cancel`

---

## Delete a package

A package can only be deleted if its linked domain is in the `deleted` state, or if the package has no domain linked to it.

```
DELETE /api/v1/packages/{package-guid}.json
```

**Response:**

```
204 No Content
```
