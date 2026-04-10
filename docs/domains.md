# Domains

A domain represents a customer's email domain. It must be linked to at least one package before it can be provisioned.

Domain state is computed from the state of its underlying service components — it is not stored directly. See [State Reference](state-reference.md).

---

## Get a domain

```
GET /api/v1/domains/{domain-guid}.json
```

Returns the full domain record including current state, service fields, and usage data (if a usage action has been run).

---

## List domains

```
GET /api/v1/domains.json
```

Common filters:

| Parameter      | Description |
|----------------|-------------|
| `domain_name`  | Filter by domain name |
| `exact_match`  | `1` to require an exact match |
| `ou_guid`      | Filter by company GUID |
| `page`         | Page number |
| `limit`        | Items per page |

---

## Create a domain

### Standalone domain (link separately)

Creates a domain under a company without linking it to a package:

```
POST /api/v1/ous/{company-guid}/domains.json
```

```json
{
  "domain": {
    "domain_name": "acme.com"
  }
}
```

**Response:**

```
201 Created
Location: /api/v1/domains/{domain-guid}
```

### Create and link to a package in one step

Creates the domain and immediately links it to the specified package. This is the recommended approach for new domains:

```
POST /api/v1/packages/{package-guid}/domains.json
```

```json
{
  "domain": {
    "domain_name": "acme.com"
  }
}
```

**Response:**

```
201 Created
Location: /api/v1/domains/{domain-guid}
```

---

## Link a domain to a package

Before a domain can be provisioned, it must be linked to at least one package. Linking determines which backend services will be provisioned.

```
LINK /api/v1/packages/{package-guid}/domains/{domain-guid}.json
```

**Response for a standard link:**

```
204 No Content
```

**Response when migration is triggered (services being decommissioned):**

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

The returned action is the automated `Prune` action for services no longer needed after the migration. Poll it before proceeding.

---

## Unlink a domain from a package

```
UNLINK /api/v1/packages/{package-guid}/domains/{domain-guid}.json
```

Only supported for inactive domains. See [Migration Guide](migration-guide.md) for the recommended way to switch products.

---

## Check migration eligibility

Performs a dry run of the LINK operation without making changes. Returns `204 No Content` if the link would succeed, or an error response if not:

```
OPTIONS /api/v1/packages/{package-guid}/domains/{domain-guid}.json
```

For migrations between products with mailbox editions, a `200 OK` response includes a mailbox edition mapping report. See [Migration Guide](migration-guide.md).

---

## Configuring service fields

Before a domain can be provisioned, it must be configured with product-specific service data. Required fields depend on the linked package(s).

**Step 1 — discover required fields:**

```
GET /api/v1/domains/{domain-guid}.json
```

Inspect the `fields` object in the response:

```json
{
  "guid": "domain-guid",
  "domain_name": "acme.com",
  "fields": {
    "mail_delivery_location": null,
    "mail_delivery_alternative": null,
    "auth_method": "smtp",
    "username": null,
    "password": null,
    "admin_username": null,
    "admin_password": null,
    "admin_first_name": null,
    "admin_last_name": null,
    "service_provider": "other"
  }
}
```

**Step 2 — populate and submit:**

```
PATCH /api/v1/domains/{domain-guid}/servicefields.json
```

```json
{
  "fields": {
    "mail_delivery_location": "smtp.acme.com",
    "mail_delivery_alternative": "smtp2.acme.com",
    "auth_method": "smtp",
    "username": "smtp-user",
    "password": "smtpPassw0rd!",
    "smtp_username": "smtp-user",
    "smtp_password": "smtpPassw0rd!",
    "admin_username": "jane.doe@acme.com",
    "admin_password": "adminPassw0rd!",
    "admin_first_name": "Jane",
    "admin_last_name": "Doe"
  }
}
```

> **Legacy compatibility:** Where endpoints expose both `smtp_username`/`smtp_password` and `username`/`password`, populate both pairs with the same values.

**Response:**

```
204 No Content
```

If the domain was already active when you patched service fields, it will enter the `stale` state and requires a `Refresh` action — see below.

---

## Domain actions

All domain actions use the same endpoint and follow the standard [async action](async-actions.md) pattern:

```
POST /api/v1/domains/{domain-guid}/actions.json
```

### Provision

Provisions the domain on all backend services linked via its packages. Must be called after service fields are configured.

```json
{
  "action": { "action": "Provision" }
}
```

### Suspend

Temporarily suspends mail flow for the domain. Mail is rejected while suspended.

```json
{
  "action": { "action": "Suspend" }
}
```

### Activate (reactivate)

Returns a closed or suspended domain to active service.

```json
{
  "action": { "action": "Activate" }
}
```

### Close

A more severe suspension. Closes the domain and rejects all incoming mail.

```json
{
  "action": { "action": "Close" }
}
```

### Refresh

Applies updated service field configuration to an already-provisioned (active or stale) domain.

```json
{
  "action": { "action": "Refresh" }
}
```

Call this after patching service fields on an active domain. When the action finishes, the domain returns to `active`.

### Update

Pushes any pending configuration changes to backend services.

```json
{
  "action": { "action": "Update" }
}
```

### Delete

Deprovisions the domain from backend services. After the action completes, the domain will be in the `deleted` state and eligible for final removal.

```json
{
  "action": { "action": "Delete" }
}
```

> **Note:** For products that expose mailboxes, all mailboxes must be deleted before a domain delete action will be accepted.

### Usage

Triggers a usage report for the domain. After the action completes, the usage data is available on the domain GET endpoint.

```json
{
  "action": { "action": "Usage" }
}
```

See [Usage Reporting](usage-reporting.md).

### Valid actions

`Provision`, `Delete`, `Purge`, `AppUsage`, `Activate`, `Close`, `Update`, `Suspend`, `Move`, `Token`, `Usage`, `Prune`, `Refresh`

All actions respond with:

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

---

## Delete a domain record

After a domain is in the `deleted` or `inactive` state, the API record can be removed:

```
DELETE /api/v1/domains/{domain-guid}.json
```

**Response:**

```
204 No Content
```

> The `DELETE` endpoint is only permitted for domains in the `inactive` or `deleted` state.

---

## Domain aliases

A domain alias is an alternative domain name that delivers to the same mailboxes as the primary domain.

**Create an alias:**

```
POST /api/v1/domains/{domain-guid}/aliases.json
```

```json
{
  "alias": {
    "domain_name": "acme.net"
  }
}
```

**Delete an alias:**

```
DELETE /api/v1/domains/{domain-guid}/aliases/{alias-domain-name}.json
```

---

## Account lock

Inspect account lock state:

```
GET /api/v1/domains/{domain-guid}/accountlock.json
```

Override account lock state (support use only):

```
PATCH /api/v1/domains/{domain-guid}/accountlock.json
```

```json
{
  "accountlock": {
    "locked": false
  }
}
```
