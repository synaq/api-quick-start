# Error Handling

---

## HTTP status codes

| Code  | Meaning |
|-------|---------|
| `200 OK` | Request succeeded with a response body |
| `201 Created` | Entity created; `Location` header contains the new resource URL |
| `202 Accepted` | Async action queued; `Location` header points to the action to poll |
| `204 No Content` | Request succeeded; no response body |
| `400 Bad Request` | Invalid request payload or missing required fields |
| `401 Unauthorized` | Missing or invalid authentication headers |
| `403 Forbidden` | Authenticated but not authorised for the requested resource |
| `404 Not Found` | Resource does not exist (also returned after a successful mailbox delete) |
| `409 Conflict` | Request rejected due to current entity state (e.g. migration blocked by active mailboxes) |
| `422 Unprocessable Entity` | Request was well-formed but semantically invalid |
| `500 Internal Server Error` | Server-side error; contact SYNAQ support |

---

## Error response format

Most error responses return a JSON body:

```json
{
  "code": 409,
  "message": "This migration will result in the removal of a service on which active mailboxes depend. The mailboxes must be removed first.",
  "errors": null
}
```

For validation errors, the `errors` field may contain an array of field-level messages:

```json
{
  "code": 400,
  "message": "Validation failed",
  "errors": [
    "email_local: This value should not be blank.",
    "last_name: This value should not be blank."
  ]
}
```

---

## Async action errors

When an async action fails, it reaches the `error` state and the action's `errors` array contains one or more messages:

```json
{
  "id": 1234,
  "action": "provision",
  "state": "error",
  "errors": [
    "Domain already provisioned on Zimbra under a different Reseller and Company"
  ]
}
```

Failed actions are **never retried automatically**. Once the underlying issue is resolved, resubmit the same action.

---

## Common issues

### Domain stuck in `pending`

The domain shows `pending` with no active jobs. Most common causes:
- An underlying service action was interrupted and not completed
- A backend job failed silently

Contact SYNAQ support with the domain GUID and the time the issue started.

### `409 Conflict` on migration link

There are active mailboxes on the domain that depend on a service not included in the new product. Delete all active mailboxes before retrying the migration.

### `404 Not Found` when polling a mailbox delete action

This is expected and indicates success. The delete action removes itself on completion. Treat `404` as `finished` for this operation only.

### Auth headers missing or invalid

Ensure:
- `synaq-api-user` matches the username exactly (case-sensitive)
- `synaq-api-salt` is a unique timestamp (do not reuse values)
- `synaq-api-hash` is SHA1 of `username + apiKey + salt` (concatenated, no separators)

### `Location` header not found

Some legacy endpoints return a lowercase `location` header. Check for both `Location` and `location` when handling `202 Accepted` responses.
