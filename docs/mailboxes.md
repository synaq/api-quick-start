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

See [Pagination](pagination.md) for the full response envelope format.

---

## Retrieving all mailboxes for a domain

### Recommended approach: filter by domain GUID

Use `domain_guid` rather than `domain_name` wherever possible. A GUID lookup is a direct index hit; a name lookup requires an additional resolution step.

```
GET /api/v1/mailboxes.json?domain_guid={domain-guid}&exact_match=1&limit=100
```

Always set `exact_match=1` to avoid partial matches from domains whose names contain yours as a substring.

Set `limit` explicitly — the default is 5 items per page, which is inefficient for domains with many mailboxes.

### Paginating through a large mailbox list

Check the `pages` field in the response to determine how many pages exist, then iterate:

**Sample response envelope:**

```json
{
  "page": 1,
  "limit": 100,
  "pages": 3,
  "total": 243,
  "_embedded": {
    "mailboxes": [ ... ]
  }
}
```

**Example — fetch all mailboxes for a domain:**

**PHP**

```php
function getAllMailboxes(string $domainGuid, array $headers, string $baseUrl): array {
    $mailboxes = [];
    $page = 1;
    do {
        $params = http_build_query([
            'domain_guid' => $domainGuid,
            'exact_match' => 1,
            'limit'       => 100,
            'page'        => $page,
        ]);
        $response = httpGet("{$baseUrl}/api/v1/mailboxes.json?{$params}", $headers); // use your HTTP client of choice
        $mailboxes = array_merge($mailboxes, $response['_embedded']['mailboxes']);
        $page++;
    } while ($page <= $response['pages']);
    return $mailboxes;
}
```

**Python**

```python
def get_all_mailboxes(domain_guid, headers, base_url):
    mailboxes = []
    page = 1
    while True:
        params = {
            'domain_guid': domain_guid,
            'exact_match': 1,
            'limit': 100,
            'page': page,
        }
        response = requests.get(
            f'{base_url}/api/v1/mailboxes.json',
            params=params,
            headers=headers
        ).json()
        mailboxes.extend(response['_embedded']['mailboxes'])
        if page >= response['pages']:
            break
        page += 1
    return mailboxes
```

**JavaScript (Node.js)**

```javascript
async function getAllMailboxes(domainGuid, headers, baseUrl) {
    const mailboxes = [];
    let page = 1;
    do {
        const params = new URLSearchParams({ domain_guid: domainGuid, exact_match: 1, limit: 100, page }).toString();
        const res = await fetch(`${baseUrl}/api/v1/mailboxes.json?${params}`, { headers });
        const response = await res.json();
        mailboxes.push(...response._embedded.mailboxes);
        page++;
    } while (page <= response.pages);
    return mailboxes;
}
```

### Retrieving a single known mailbox by email address

When you know the full email address:

```
GET /api/v1/mailboxes.json?domain_name=acme.com&email_local=jane.doe&exact_match=1
```

This will return a single-item result set. Extract the GUID from `_embedded.mailboxes[0].guid` for subsequent operations.

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

**PHP**

```php
function ssha256(string $password): string {
    $salt   = random_bytes(4);
    $digest = hash('sha256', $password . $salt, true);
    return '{SSHA256}' . base64_encode($digest . $salt);
}
```

**Python**

```python
import os
import hashlib
import base64

def ssha256(password: str) -> str:
    salt = os.urandom(4)
    digest = hashlib.sha256(password.encode() + salt).digest()
    return '{SSHA256}' + base64.b64encode(digest + salt).decode()
```

**JavaScript (Node.js)**

```javascript
const crypto = require('crypto');

function ssha256(password) {
    const salt   = crypto.randomBytes(4);
    const digest = crypto.createHash('sha256').update(Buffer.concat([Buffer.from(password), salt])).digest();
    return '{SSHA256}' + Buffer.concat([digest, salt]).toString('base64');
}
```

---

## Quota usage

Returns the current storage used by the mailbox. Only available for active mailboxes.

```
GET /api/v1/mailboxes/{mailbox-guid}/quota.json
```

**Sample response:**

```json
{
  "quota_used": "524288000"
}
```

`quota_used` is returned as a string, in bytes.

---

## Folder statistics

Returns per-folder message counts and sizes. Only available for active mailboxes.

```
GET /api/v1/mailboxes/{mailbox-guid}/folderstats.json
```

**Sample response:**

```json
{
  "folder_stats": [
    {
      "name": "Inbox",
      "message_count": 142,
      "size": 10485760
    },
    {
      "name": "Sent",
      "message_count": 37,
      "size": 2097152
    }
  ]
}
```

---

## Whitelists

Manage inbound sender whitelists for a mailbox. Each entry matches by address, domain, IPv4, or CIDR range.

### List whitelist entries

```
GET /api/v1/mailboxes/{mailbox-guid}/whitelists.json
```

| Parameter | Description |
|-----------|-------------|
| `alias`   | Filter entries belonging to a specific alias address |
| `page` / `limit` | Pagination |

**Sample response:**

```json
{
  "page": 1,
  "limit": 25,
  "pages": 1,
  "total": 2,
  "_embedded": {
    "whitelists": [
      {
        "api_id": "wl-abc123",
        "direction": "in",
        "from_filter_value": "trusted@example.com"
      }
    ]
  }
}
```

### Add a whitelist entry

```
POST /api/v1/mailboxes/{mailbox-guid}/whitelists.json
```

```json
{
  "list": {
    "from_filter_type": "address",
    "from_filter_value": "trusted@example.com",
    "alias": "alias@acme.com"
  }
}
```

| Field | Description |
|-------|-------------|
| `from_filter_type` | `address`, `domain`, `ipv4`, or `cidr` |
| `from_filter_value` | The value to match |
| `alias` | Optional — scope the entry to a specific alias of the mailbox |

**Response:**

```
204 No Content
```

### Replace all whitelist entries

Replaces the entire whitelist in one operation.

```
PATCH /api/v1/mailboxes/{mailbox-guid}/whitelists.json
```

```json
{
  "mailbox": {
    "access_lists": [
      { "from_filter_type": "domain", "from_filter_value": "trusted.com" },
      { "from_filter_type": "address", "from_filter_value": "contact@partner.com" }
    ]
  }
}
```

**Response:**

```
204 No Content
```

### Remove a whitelist entry

```
DELETE /api/v1/mailboxes/{mailbox-guid}/whitelists/{api-id}.json
```

**Response:**

```
204 No Content
```

---

## Blacklists

Manage inbound sender blacklists for a mailbox. Follows the same structure as whitelists.

### List blacklist entries

```
GET /api/v1/mailboxes/{mailbox-guid}/blacklists.json
```

Supports the same `alias`, `page`, and `limit` parameters as whitelists.

### Add a blacklist entry

```
POST /api/v1/mailboxes/{mailbox-guid}/blacklists.json
```

```json
{
  "list": {
    "from_filter_type": "domain",
    "from_filter_value": "spam-source.com"
  }
}
```

**Response:**

```
204 No Content
```

### Replace all blacklist entries

```
PATCH /api/v1/mailboxes/{mailbox-guid}/blacklists.json
```

```json
{
  "mailbox": {
    "access_lists": [
      { "from_filter_type": "domain", "from_filter_value": "spam-source.com" },
      { "from_filter_type": "cidr", "from_filter_value": "192.0.2.0/24" }
    ]
  }
}
```

**Response:**

```
204 No Content
```

### Remove a blacklist entry

```
DELETE /api/v1/mailboxes/{mailbox-guid}/blacklists/{api-id}.json
```

**Response:**

```
204 No Content
```

---

## Signatures

Set the primary and secondary signatures for a mailbox. Only available for `inactive` mailboxes — configure signatures before provisioning.

```
PATCH /api/v1/mailboxes/{mailbox-guid}/signatures.json
```

```json
{
  "mailbox": {
    "primary_signature": "Regards,\nJane Doe",
    "secondary_signature": "Sent from mobile",
    "preferred_signature": "primary"
  }
}
```

| Field | Description |
|-------|-------------|
| `primary_signature` | Main signature text |
| `secondary_signature` | Alternate signature text |
| `preferred_signature` | Which signature to use by default: `primary` or `secondary` |

**Response:**

```
204 No Content
```

---

## Access statistics

Returns a paginated log of mailbox login and access events within a time range.

```
GET /api/v1/mailboxes/{mailbox-guid}/accessstats.json
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `start_time` | Yes | Start of the time range, ISO 8601 UTC — e.g. `2024-01-01T00:00:00Z` |
| `end_time` | No | End of the time range. Defaults to now |
| `methods` | No | Comma-separated access methods to include. Defaults to all: `pop3,imap,webmail,zsync,soap,zco` |
| `page` / `limit` | No | Pagination |

**Sample request:**

```
GET /api/v1/mailboxes/{mailbox-guid}/accessstats.json?start_time=2024-01-01T00:00:00Z&methods=imap,webmail
```

**Sample response:**

```json
{
  "page": 1,
  "limit": 25,
  "pages": 1,
  "total": 3,
  "_embedded": {
    "records": [
      {
        "timestamp": "2024-01-15T08:23:11Z",
        "method": "imap",
        "ip_address": "203.0.113.42",
        "user_agent": "Thunderbird/115"
      }
    ]
  }
}
```

---

## Mail statistics

Four endpoints return paginated mail traffic records for a mailbox. All follow the same parameter structure and response envelope.

### Parameters (all four endpoints)

| Parameter | Required | Description |
|-----------|----------|-------------|
| `start_time` | Yes | Start of the time range, ISO 8601 UTC — e.g. `2024-01-01T00:00:00Z` |
| `end_time` | No | End of the time range. Defaults to now |
| `direction` | No | Comma-separated: `inbound`, `outbound`, or both (default) |
| `page` / `limit` | No | Pagination |

### Clean mail

Messages that passed all filters and were delivered.

```
GET /api/v1/mailboxes/{mailbox-guid}/cleanmailstats.json
```

### Junk mail

Messages flagged as spam.

```
GET /api/v1/mailboxes/{mailbox-guid}/junkmailstats.json
```

Junk mail also supports an optional `recipient` parameter to filter outbound junk by recipient address.

### Rejected mail

Messages rejected at the gateway before delivery.

```
GET /api/v1/mailboxes/{mailbox-guid}/rejectedmailstats.json
```

### Virus mail

Messages blocked due to virus detection.

```
GET /api/v1/mailboxes/{mailbox-guid}/virusmailstats.json
```

**Sample response (all four endpoints):**

```json
{
  "page": 1,
  "limit": 25,
  "pages": 2,
  "total": 41,
  "_embedded": {
    "records": [
      {
        "timestamp": "2024-01-15T09:10:00Z",
        "from": "sender@example.com",
        "to": "jane@acme.com",
        "subject": "Meeting notes",
        "direction": "inbound",
        "size": 14823
      }
    ]
  }
}
```
