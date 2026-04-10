# Calendar Resources

A calendar resource is an email address that acts as a bookable placeholder for a shared location or piece of equipment (meeting room, projector, etc.). Available on products that include a mailbox service.

---

## Search / list calendar resources

```
GET /api/v1/resources.json?domain_guid={domain-guid}&exact_match=1
```

Supports the same filters and pagination as the mailboxes endpoint. See [Pagination](pagination.md).

---

## Get a calendar resource

```
GET /api/v1/resources/{resource-guid}.json
```

---

## Fields reference

| Field           | Required | Description |
|-----------------|----------|-------------|
| `email_local`   | Yes | Local part of the resource's email address |
| `password`      | Yes* | Plain text password (hashed internally on receipt) |
| `ssha_password` | Yes* | SSHA256 pre-hashed password — use instead of `password` for better security |
| `display_name`  | Yes | Display name shown in the global address list and meeting responses |
| `resource_type` | Yes | `Location` or `Equipment` |

*Provide either `password` or `ssha_password`.

---

## Create a calendar resource

```
POST /api/v1/domains/{domain-guid}/resources.json
```

```json
{
  "resource": {
    "email_local": "boardroom-a",
    "password": "SomePassw0rd$",
    "display_name": "Boardroom A",
    "resource_type": "Location"
  }
}
```

**Response:**

```
201 Created
Location: /api/v1/resources/{resource-guid}
```

---

## Update a calendar resource

For active resources, an async `Sync` action is automatically created.

```
PATCH /api/v1/resources/{resource-guid}.json
```

Include only the fields to change:

```json
{
  "resource": {
    "display_name": "Boardroom A (Level 3)",
    "password": "NewPassw0rd!"
  }
}
```

**Response for inactive resources:**

```
204 No Content
```

**Response for active resources:**

```
202 Accepted
Location: /api/v1/resources/{resource-guid}/actions/{action-id}
```

---

## Resource actions

All resource actions follow the standard [async action](async-actions.md) pattern:

```
POST /api/v1/resources/{resource-guid}/actions.json
```

### Provision

```json
{ "action": { "action": "Provision" } }
```

### Delete (from backend services)

Active resources must be deleted via an action before the record can be removed:

```json
{ "action": { "action": "Delete" } }
```

Once the action completes, the resource record is automatically removed from the API.

Valid resource actions: `Provision`, `Delete`, `Sync`

---

## Delete an inactive calendar resource

If a resource was never provisioned, the record can be removed directly:

```
DELETE /api/v1/resources/{resource-guid}.json
```

**Response:**

```
204 No Content
```
