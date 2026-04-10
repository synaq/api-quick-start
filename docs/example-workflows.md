# Example Workflows

End-to-end examples for the most common integration scenarios. Each workflow shows every API call in sequence, including polling steps.

For reference:
- [Authentication](authentication.md)
- [Async Actions](async-actions.md) — polling pattern used throughout
- [State Reference](state-reference.md)

---

## Before you begin: your Reseller GUID

Every reseller on the SYNAQ platform has a unique **Reseller GUID** — a UUID that identifies your top-level account. It is required whenever you create a new customer company, and is the starting point for your entire entity hierarchy.

Your Reseller GUID is provided by SYNAQ when your API credentials are provisioned. It will be included in the onboarding information you receive alongside your API username and key.

**If you do not have your Reseller GUID**, you can look it up via the API once authenticated:

```
GET /api/v1/ous.json?type=reseller&limit=1
```

Your reseller will appear as the top-level result. Extract the `guid` field:

```json
{
  "page": 1,
  "limit": 1,
  "pages": 1,
  "total": 1,
  "_embedded": {
    "ous": [
      {
        "guid": "{your-reseller-guid}",
        "title": "Your Reseller Name",
        "type": "reseller"
      }
    ]
  }
}
```

Store this GUID in your integration's configuration. All new customer companies are created under it.

---

## Workflow 1: New customer on a mailbox product

Covers creating a company, package, domain, and mailbox from scratch.

### Step 1 — Create the company

```
POST /api/v1/ous/{reseller-guid}/companies.json
```

```json
{
  "ou": {
    "title": "Acme Ltd",
    "physical_address": {
      "line_1": "20 Long Street",
      "city": "Johannesburg",
      "postal_code": "2001",
      "country": "ZA"
    }
  }
}
```

```
201 Created
Location: /api/v1/ous/{company-guid}
```

Extract `{company-guid}` from the `Location` header.

---

### Step 2 — Create the package

```
POST /api/v1/ous/{company-guid}/packages.json
```

```json
{
  "package": {
    "code": "{package-code}"
  }
}
```

```
201 Created
Location: /api/v1/packages/{package-guid}
```

---

### Step 3 — Create the domain and link it to the package

Using the convenience endpoint that creates and links in one step:

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

```
201 Created
Location: /api/v1/domains/{domain-guid}
```

---

### Step 4 — Discover and configure service fields

First, inspect which fields the product requires:

```
GET /api/v1/domains/{domain-guid}.json
```

Inspect the `fields` object in the response. Then submit the populated fields:

```
PATCH /api/v1/domains/{domain-guid}/servicefields.json
```

```json
{
  "fields": {
    "mail_delivery_location": "mail.acme.com",
    "auth_method": "smtp",
    "username": "smtp-user",
    "password": "smtpPassw0rd!",
    "smtp_username": "smtp-user",
    "smtp_password": "smtpPassw0rd!",
    "admin_username": "admin@acme.com",
    "admin_password": "adminPassw0rd!",
    "admin_first_name": "Jane",
    "admin_last_name": "Doe"
  }
}
```

```
204 No Content
```

> The exact fields required depend on the product. Fetch the domain first and populate only the fields shown in the `fields` object.

---

### Step 5 — Provision the domain

```
POST /api/v1/domains/{domain-guid}/actions.json
```

```json
{
  "action": {
    "action": "Provision"
  }
}
```

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

**Poll until finished:**

```
GET /api/v1/domains/{domain-guid}/actions/{action-id}.json
```

Wait for `"state": "finished"`. The domain is now `active`.

---

### Step 6 — Create a mailbox

```
POST /api/v1/domains/{domain-guid}/mailboxes.json
```

```json
{
  "mailbox": {
    "email_local": "jane.doe",
    "password": "MailboxPassw0rd!",
    "last_name": "Doe",
    "first_name": "Jane",
    "display_name": "Jane Doe",
    "package_edition_code": "{edition-code}"
  }
}
```

```
201 Created
Location: /api/v1/mailboxes/{mailbox-guid}
```

> Edition codes are available by GETting the package. If omitted, the default edition for the package is used.

---

### Step 7 — Provision the mailbox

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

```json
{
  "action": {
    "action": "Provision"
  }
}
```

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

**Poll until finished:**

```
GET /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}.json
```

Wait for `"state": "finished"`. The mailbox is now `active`.

> **Shortcut:** Steps 6 and 7 can be combined by setting `"provision_immediately": true` on the mailbox creation payload. The response will be `202 Accepted` with a reference to the provision action.

---

## Workflow 2: New customer on a non-mailbox product

For products that do not expose mailboxes via the API (e.g. email security products), the workflow ends after domain provisioning.

### Step 1 — Create the company

Same as Workflow 1, Step 1.

---

### Step 2 — Create the package

```
POST /api/v1/ous/{company-guid}/packages.json
```

```json
{
  "package": {
    "code": "{package-code}"
  }
}
```

```
201 Created
Location: /api/v1/packages/{package-guid}
```

---

### Step 3 — Create the domain and link it

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

```
201 Created
Location: /api/v1/domains/{domain-guid}
```

---

### Step 4 — Configure service fields

```
GET /api/v1/domains/{domain-guid}.json
```

Inspect `fields`, then submit:

```
PATCH /api/v1/domains/{domain-guid}/servicefields.json
```

```json
{
  "fields": {
    "mail_delivery_location": "mail.acme.com",
    "mail_delivery_alternative": "mail2.acme.com",
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

```
204 No Content
```

---

### Step 5 — Provision the domain

```
POST /api/v1/domains/{domain-guid}/actions.json
```

```json
{
  "action": {
    "action": "Provision"
  }
}
```

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

**Poll until finished:**

```
GET /api/v1/domains/{domain-guid}/actions/{action-id}.json
```

Wait for `"state": "finished"`. The domain is now `active`. No further steps are required.

---

## Workflow 3: Suspension

Use suspension as a temporary, reversible measure — for example, to enforce credit control without removing the customer's data.

### Suspending a domain

Suspending a domain blocks mail flow for the entire domain.

```
POST /api/v1/domains/{domain-guid}/actions.json
```

```json
{
  "action": {
    "action": "Suspend"
  }
}
```

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

Poll until `finished`. The domain enters the `suspended` state.

### Suspending a mailbox (mailbox products only)

Individual mailboxes can be suspended independently of the domain. A suspended mailbox still receives mail but the end user cannot log in.

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

```json
{
  "action": {
    "action": "Suspend"
  }
}
```

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

### Reactivating a suspended domain or mailbox

```
POST /api/v1/domains/{domain-guid}/actions.json
```

```json
{
  "action": {
    "action": "Activate"
  }
}
```

Same pattern for mailboxes, using the mailbox actions endpoint. Poll until `finished`.

---

## Workflow 4: Closing

Closing is a more severe form of service suspension. A closed domain rejects all incoming mail. A closed mailbox is inaccessible and mail destined for it bounces. Data is preserved in both cases.

### Closing a domain

```
POST /api/v1/domains/{domain-guid}/actions.json
```

```json
{
  "action": {
    "action": "Close"
  }
}
```

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

Poll until `finished`.

### Closing a mailbox (mailbox products only)

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

```json
{
  "action": {
    "action": "Close"
  }
}
```

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

### Reactivating a closed domain or mailbox

Use the `Activate` action on the relevant endpoint, as shown in Workflow 3.

---

## Workflow 5: Cancellation

Full removal of a customer's service. For mailbox products, mailboxes must be removed before the domain can be deprovisioned.

### Step 1 — Delete all mailboxes (mailbox products only)

For each active mailbox under the domain:

```
DELETE /api/v1/mailboxes/{mailbox-guid}.json
```

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

Poll the action. When the deletion is complete, the action record itself is removed — a `404 Not Found` response on the action URL confirms success.

Repeat for every mailbox. The domain cannot be deleted while provisioned mailboxes remain.

---

### Step 2 — Deprovision the domain

```
POST /api/v1/domains/{domain-guid}/actions.json
```

```json
{
  "action": {
    "action": "Delete"
  }
}
```

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

Poll until `finished`. The domain enters the `deleted` state.

---

### Step 3 — Remove the domain record

```
DELETE /api/v1/domains/{domain-guid}.json
```

```
204 No Content
```

> Only permitted for domains in the `deleted` or `inactive` state.

---

### Step 4 — Remove the package record

```
DELETE /api/v1/packages/{package-guid}.json
```

```
204 No Content
```

> Only permitted when the package has no domain linked to it, or its linked domain is in the `deleted` state.

---

### Step 5 — Remove the company (optional)

If the company has no remaining active packages, it can be removed:

```
DELETE /api/v1/ous/{company-guid}.json
```

```
204 No Content
```
