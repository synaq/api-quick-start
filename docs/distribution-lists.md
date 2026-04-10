# Distribution Lists

A distribution list (DL) is an email address that delivers to multiple members. Available on products that include a mailbox service.

---

## Search / list distribution lists

```
GET /api/v1/dls.json?domain_guid={domain-guid}&exact_match=1
```

Supports the same filters and pagination as the mailboxes endpoint. See [Pagination](pagination.md).

---

## Get a distribution list

```
GET /api/v1/dls/{dl-guid}.json
```

---

## Fields reference

| Field         | Required | Description |
|---------------|----------|-------------|
| `email_local` | Yes (create) | Local part of the DL's email address |
| `hide_in_gal` | No | Boolean — hide the DL from the global address list |
| `members`     | No | Array of member objects (see below) |

**Member object format:**

```json
{
  "local": "some.user",
  "domain": "some-domain.com"
}
```

Members do not have to be on the same domain as the DL.

> **Deprecated:** The `catch_all_address` field is deprecated and should not be used in new integrations.

---

## Create a distribution list

```
POST /api/v1/domains/{domain-guid}/dls.json
```

```json
{
  "dl": {
    "email_local": "team",
    "hide_in_gal": false,
    "members": [
      {
        "local": "jane.doe",
        "domain": "acme.com"
      },
      {
        "local": "john.smith",
        "domain": "acme.com"
      }
    ]
  }
}
```

**Response:**

```
201 Created
Location: /api/v1/dls/{dl-guid}
```

---

## Update a distribution list

For active DLs, an async `Sync` action is automatically created to propagate changes.

```
PATCH /api/v1/dls/{dl-guid}.json
```

Include only the fields to change. To rename a DL, change `email_local`. To update members, provide the full new member list (the API replaces the list entirely, not merges it).

```json
{
  "dl": {
    "members": [
      {
        "local": "jane.doe",
        "domain": "acme.com"
      },
      {
        "local": "new.member",
        "domain": "acme.com"
      }
    ]
  }
}
```

**Response for inactive DLs:**

```
204 No Content
```

**Response for active DLs:**

```
202 Accepted
Location: /api/v1/dls/{dl-guid}/actions/{action-id}
```

---

## DL actions

All DL actions follow the standard [async action](async-actions.md) pattern:

```
POST /api/v1/dls/{dl-guid}/actions.json
```

### Provision

```json
{ "action": { "action": "Provision" } }
```

### Delete (from backend services)

Active DLs must be deleted via an action before the record can be removed:

```json
{ "action": { "action": "Delete" } }
```

Once the action completes, the DL record is automatically removed from the API.

Valid DL actions: `Provision`, `Delete`, `Sync`

---

## Delete an inactive distribution list

If a DL was never provisioned, the record can be removed directly:

```
DELETE /api/v1/dls/{dl-guid}.json
```

**Response:**

```
204 No Content
```
