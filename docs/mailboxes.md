# Mailboxes

Mailboxes are individual email accounts under a provisioned domain. Only available for products that include a mailbox service.

---

## Get a mailbox

```
GET /api/v1/mailboxes/{mailbox-guid}.json
```

---

## Search / list mailboxes

```
GET /api/v1/mailboxes.json
```

Common filters:

| Parameter        | Description |
|------------------|-------------|
| `domain_guid`    | Filter by domain GUID |
| `domain_name`    | Filter by domain name |
| `email_local`    | Filter by local part of address |
| `exact_match`    | `1` for exact matches only (recommended) |
| `sort_descending`| `1` to reverse sort order |
| `page` / `limit` | Pagination |

```
GET /api/v1/mailboxes.json?domain_name=acme.com&exact_match=1&limit=25
```

See [Pagination](pagination.md) for the full response envelope format.

---

## Create a mailbox

```
POST /api/v1/domains/{domain-guid}/mailboxes.json
```

**Required fields:** `email_local`, `password` (or `ssha_password`), `last_name`

```json
{
  "mailbox": {
    "email_local": "jane.doe",
    "password": "Sample123$",
    "last_name": "Doe",
    "first_name": "Jane",
    "display_name": "Jane Doe",
    "package_edition_code": "{edition-code}"
  }
}
```

Specifying `package_edition_code` is recommended — it assigns the mailbox to a specific edition (storage tier) immediately. Valid edition codes can be found by GETting the package.

**Response:**

```
201 Created
Location: /api/v1/mailboxes/{mailbox-guid}
```

---

## Create and provision immediately

Raises the `provision_immediately` flag to skip the separate provision step. The API responds with a reference to the auto-created provision action:

```
POST /api/v1/domains/{domain-guid}/mailboxes.json
```

```json
{
  "mailbox": {
    "email_local": "jane.doe",
    "password": "Sample123$",
    "last_name": "Doe",
    "package_edition_code": "{edition-code}",
    "provision_immediately": true
  }
}
```

**Response:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

The mailbox GUID is embedded in the action location path.

---

## Update a mailbox

```
PATCH /api/v1/mailboxes/{mailbox-guid}.json
```

Include only the fields to change. For active mailboxes, an async `Update` action is automatically created.

```json
{
  "mailbox": {
    "password": "NewPassw0rd!",
    "display_name": "Jane Smith"
  }
}
```

**Response for inactive mailboxes:**

```
204 No Content
```

**Response for active mailboxes:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

---

## Delete a mailbox

Automatically creates a delete action. For active mailboxes, the delete is asynchronous.

```
DELETE /api/v1/mailboxes/{mailbox-guid}.json
```

**Response for inactive mailboxes:**

```
204 No Content
```

**Response for active mailboxes:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

> **Special case:** A successful mailbox delete removes the action record itself. Poll the action until you receive `404 Not Found` — that indicates the deletion is complete. See [Async Actions](async-actions.md).

---

## Mailbox actions

All mailbox actions use the same endpoint and follow the standard [async action](async-actions.md) pattern:

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

### Provision

```json
{ "action": { "action": "Provision" } }
```

### Suspend

Blocks user access. Mail is still received.

```json
{ "action": { "action": "Suspend" } }
```

### Activate

Returns a suspended or closed mailbox to active service.

```json
{ "action": { "action": "Activate" } }
```

### Close

More severe than suspend. Blocks access and rejects incoming mail. Data is preserved.

```json
{ "action": { "action": "Close" } }
```

### Unlock

Used to unlock a mailbox that SYNAQ has locked due to abuse (account compromise, spam, etc.). Change the password before unlocking to prevent immediate re-lock.

```json
{ "action": { "action": "Unlock" } }
```

Inspecting a locked mailbox:

```json
{
  "guid": "mailbox-guid",
  "email": "jane@acme.com",
  "state": "suspended",
  "account_locked": true,
  "lock_type": "full",
  "lock_reason_code": "ABC123",
  "lock_reason": "Human-readable reason"
}
```

### Token (generate auth token)

Generates a time-limited single-sign-on token for direct webmail login without exposing credentials. Only valid for `active` mailboxes.

```json
{ "action": { "action": "Token" } }
```

Once generated, retrieve the token:

```
GET /api/v1/mailboxes/{mailbox-guid}/token.json
```

**Response if token exists:**

```json
{
  "authentication_token": "EXAMPLE_TOKEN",
  "authentication_token_expiry": "2025-10-10T10:10:00+0200"
}
```

**Response if no valid token:**

```
204 No Content
```

Redirect the user's browser to:

```
https://{webmail-hostname}/?zauthtoken={token}
```

> **Security:** Restrict token generation to trusted staff. Keep an audit log. Enforce HTTPS for all webmail redirects. Never store the token itself — only cache the expiry date.

Valid mailbox actions: `Provision`, `Suspend`, `Activate`, `Import`, `AddContacts`, `Purge`, `Sync`, `ForceMove`, `Close`, `Unlock`, `Token`

---

## Change mailbox edition (class of service)

Upgrades or downgrades the mailbox storage tier and feature set. For active mailboxes, automatically creates an async update action.

```
PATCH /api/v1/mailboxes/{mailbox-guid}/packages/{package-guid}/edition.json
```

```json
{
  "edition": {
    "code": "{edition-code}"
  }
}
```

Valid edition codes are available by GETting the package.

---

## Check DL membership

Returns all distribution lists the mailbox belongs to (same company, active mailboxes only):

```
GET /api/v1/mailboxes/{mailbox-guid}/membership.json
```

**Sample response:**

```json
{
  "guid": "mailbox-guid",
  "email": "jane@acme.com",
  "dls": [
    {
      "guid": "dl-guid",
      "email": "team@acme.com",
      "state": "active"
    }
  ]
}
```

---

## Add email aliases

A mailbox alias is an additional email address that delivers to the same mailbox.

**Add an alias:**

```
POST /api/v1/mailboxes/{mailbox-guid}/aliases.json
```

```json
{
  "alias": {
    "address": "j.doe@acme.com"
  }
}
```

**Remove an alias:**

```
DELETE /api/v1/mailboxes/{mailbox-guid}/aliases/{alias-address}.json
```

---

## Hashed passwords

All password endpoints accept either a plain text `password` field (hashed internally on receipt) or a pre-hashed `ssha_password` field. Providing a pre-hashed password is more secure because the plain text password never leaves your system.

Supported formats: `{SSHA256}` (recommended) or `{SSHA}`

**Generating a salted SHA-256 hash:**

```
salt = four_secure_random_bytes()
ssha_password = "{SSHA256}" + base64( sha256(password + salt) + salt )
```

**Generating a salted SHA-1 hash:**

```
salt = four_secure_random_bytes()
ssha_password = "{SSHA}" + base64( sha1(password + salt) + salt )
```

```python
import os
import hashlib
import base64

def ssha256(password: str) -> str:
    salt = os.urandom(4)
    digest = hashlib.sha256(password.encode() + salt).digest()
    return '{SSHA256}' + base64.b64encode(digest + salt).decode()
```
