# State Reference

Entity states in the SYNAQ API indicate the current condition of a domain, mailbox, package, distribution list, or calendar resource. For domains and packages, state is **computed** at request time from underlying service states — it is not stored in a field.

---

## State hierarchy

When a domain or package has multiple underlying service components, the API uses this priority order (highest first) to determine the displayed state:

| Priority | State       | Meaning |
|----------|-------------|---------|
| 1        | `orphaned`  | Entity is in an inconsistent state with no viable recovery path; requires SYNAQ support |
| 2        | `stale`     | Configuration has changed and backend services need to be refreshed — use the `Refresh` action |
| 3        | `error`     | The most recent action failed; inspect action errors for details |
| 4        | `failed`    | A provisioning or deprovisioning attempt has permanently failed |
| 5        | `pending`   | One or more underlying services are awaiting completion of an async action |
| 6        | `inactive`  | Created in the API but not yet provisioned on backend services |
| 7        | `active`    | Fully provisioned and operational |

> **Important:** A single service component in `pending` state causes the entire domain or package to show `pending`. This is by design — the overall state reflects the least-ready component.

---

## Additional states

These states appear on specific entity types:

| State       | Applies to       | Meaning |
|-------------|------------------|---------|
| `suspended` | Mailboxes        | Access is blocked but mail is still received |
| `closed`    | Domains, mailboxes | Service is halted; mail delivery may be rejected |
| `deleted`   | Domains, mailboxes | Successfully deprovisioned from backend; eligible for API record deletion |
| `cancelled` | Packages         | Package has been cancelled via the `Cancel` action |
| `new`       | Actions          | Action is queued but not yet processing |

---

## What to do when stuck in `pending`

If an entity shows `pending` and has been that way for more than a few minutes with no active job:

1. Check whether there are any active async actions by inspecting recent action records.
2. Look for failed jobs on the backend — check q-services-bus logs or contact SYNAQ support.
3. If you have direct database access, the issue is most commonly an `DomainServiceAction` with `state='pending'` whose corresponding bus job has already finished.

For production debugging, contact SYNAQ support with the domain/mailbox GUID and the approximate time the issue started.

---

## Action states

Async action objects have their own state field, distinct from entity states:

| State      | Meaning |
|------------|---------|
| `new`      | Created, not yet picked up by a worker |
| `pending`  | Being processed |
| `finished` | Completed successfully |
| `error`    | Failed — see `errors` array |
