# Core Concepts

Understanding these concepts before diving into the API will save a lot of back-and-forth.

---

## Entity hierarchy

```
Reseller (you)
└── Company (your customer)
    └── Package (product instance)
        └── Domain (linked to package)
            ├── Mailbox
            ├── Distribution List
            └── Calendar Resource
```

Every entity in the API is scoped to a reseller. You can only see and manage entities that belong to your reseller or its sub-resellers.

---

## Products

A **product** is a bundle of underlying SYNAQ services sold as a unit. Your SYNAQ account manager can provide information on available products and their codes.

## Editions

Products that include mailboxes support multiple **editions** — tiers that differ in mailbox storage size, feature set, or both.

A package has one or more editions. When creating a mailbox, you specify which edition it should use. The API automatically assigns appropriate editions for new products; you do not need to specify editions when creating a package.

## Organisational Units (OUs)

An **OU** represents a customer in your hierarchy. OUs are either:

- **Companies** — direct customers. Packages and domains live under companies.
- **Sub-resellers** — intermediate resellers who can have their own companies underneath them.

See [OUs](ous.md) for CRUD operations.

## Packages

A **package** is an instance of a product under a company. One package → one domain (in the current structure). To serve multiple domains on the same product, create a separate package for each.

See [Packages](packages.md) for CRUD and provisioning.

## Domains

A **domain** represents a customer's email domain (e.g. `acme.com`). Domains must be linked to at least one package before they can be provisioned. The set of packages linked to a domain determines which backend services are active for it.

Domain state is **computed** from the state of its service components — it is not stored directly. See [State reference](state-reference.md).

See [Domains](domains.md) for all domain operations.

## Mailboxes

A **mailbox** is an individual email account under a provisioned domain. Only products that include a mailbox service expose mailboxes via the API.

See [Mailboxes](mailboxes.md) for all mailbox operations.

## Distribution Lists

A **distribution list** (DL) is a special email address that delivers to multiple members. Available on products that include a mailbox service.

See [Distribution Lists](distribution-lists.md).

## Calendar Resources

A **calendar resource** is an email address representing a bookable asset (meeting room, projector, equipment). Available on products that include a mailbox service.

See [Calendar Resources](calendar-resources.md).

---

## Service fields

Before a domain can be provisioned, it must be configured with **service fields** — product-specific settings such as mail delivery destination, SMTP authentication credentials, and admin account details. Which fields are required depends on the product(s) linked to the domain.

To discover which fields are required, GET the domain and inspect the `fields` object. Then POST the populated fields to the service fields endpoint.

See [Domains — Configuring service fields](domains.md#configuring-service-fields).

---

## Asynchronous actions

Most write operations that touch backend services are **asynchronous**. When you trigger one, the API responds with `202 Accepted` and a `Location` header pointing to the newly created action resource:

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

Your client must **poll** that location until the action reaches a terminal state:

| State      | Meaning                                         |
|------------|-------------------------------------------------|
| `pending`  | Queued or in progress                           |
| `finished` | Completed successfully                          |
| `error`    | Failed — inspect the `errors` array for details |

For the full polling workflow and error handling, see [Async Actions](async-actions.md).

---

## Domain and mailbox actions

The most common asynchronous operations are **actions** posted to a domain or mailbox actions endpoint:

```
POST /api/v1/domains/{domain-guid}/actions.json
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

Valid domain actions: `Provision`, `Delete`, `Purge`, `AppUsage`, `Activate`, `Close`, `Update`, `Suspend`, `Move`, `Token`, `Usage`, `Prune`, `Refresh`

Valid mailbox actions: `Provision`, `Suspend`, `Activate`, `Import`, `AddContacts`, `Purge`, `Sync`, `ForceMove`, `Close`, `Unlock`, `Token`

---

## Updating active domains

After a domain is provisioned, you can update its service fields at any time using the same PATCH endpoint. Once updated, the domain enters the `stale` state, indicating that the backend needs to be refreshed. Use the `Refresh` action to propagate the changes.

See [Domains — Refreshing a domain](domains.md#refreshing-a-domain).
