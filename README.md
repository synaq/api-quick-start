# SYNAQ API Documentation

Valid for release 2022-04-29.2 and later. Last updated 2025-04-10.

The SYNAQ API allows resellers to directly manage customer data and provision SYNAQ services for their customers' domains.

---

## Quick start

1. **Get credentials** — contact SYNAQ to receive your reseller GUID and API key, plus staging environment access
2. **Authenticate** — all requests require SWSS auth headers ([Authentication guide](docs/authentication.md))
3. **Create a company** — `POST /api/v1/ous/{reseller-guid}/companies.json` ([OUs](docs/ous.md))
4. **Create a package** — `POST /api/v1/ous/{company-guid}/packages.json` ([Packages](docs/packages.md))
5. **Create and link a domain** — `POST /api/v1/packages/{package-guid}/domains.json` ([Domains](docs/domains.md))
6. **Configure service fields** — `PATCH /api/v1/domains/{domain-guid}/servicefields.json`
7. **Provision** — `POST /api/v1/domains/{domain-guid}/actions.json` with `{"action":{"action":"Provision"}}`
8. **Poll the action** — `GET /api/v1/domains/{domain-guid}/actions/{action-id}.json` ([Async Actions](docs/async-actions.md))

---

## Documentation

### Foundations

| Guide | Description |
|-------|-------------|
| [Authentication](docs/authentication.md) | SWSS auth headers with PHP, Python, and Node.js examples |
| [Core Concepts](docs/concepts.md) | Entity hierarchy, products, editions, service fields, async actions |
| [Async Actions](docs/async-actions.md) | How to poll actions, handle errors, and retry failures |
| [State Reference](docs/state-reference.md) | What each entity state means and the priority hierarchy |
| [Pagination](docs/pagination.md) | HATEOAS envelope format, how to iterate all pages |
| [Error Handling](docs/error-handling.md) | HTTP status codes, error response format, common issues |

### Resources

| Guide | Description |
|-------|-------------|
| [Organisations (OUs)](docs/ous.md) | Create and manage companies and sub-resellers |
| [Packages](docs/packages.md) | Create, provision, cancel, and delete product packages |
| [Domains](docs/domains.md) | Full domain lifecycle: create, link, configure, provision, suspend, delete |
| [Mailboxes](docs/mailboxes.md) | Mailbox management, auth tokens, edition changes, hashed passwords |
| [Distribution Lists](docs/distribution-lists.md) | Create and manage email groups |
| [Calendar Resources](docs/calendar-resources.md) | Create and manage bookable resources |
| [Usage Reporting](docs/usage-reporting.md) | Trigger and retrieve billing usage reports |

### Advanced workflows

| Guide | Description |
|-------|-------------|
| [Example Workflows](docs/example-workflows.md) | Full end-to-end flows: provisioning, suspension, closing, cancellation |
| [Migration Guide](docs/migration-guide.md) | Migrate domains between products; move domains between companies |
| [Legacy API Changes](docs/legacy-changes.md) | Upgrade notes from the pre-2019 API structure |

---

## Support

Contact the SYNAQ development team at <dev@synaq.com> for integration assistance.

---

## Prerequisites

- **Reseller GUID** — your top-level identifier, provided by SYNAQ
- **API key** — provided alongside your GUID
- **Staging credentials** — provided first; production credentials issued after successful staging integration

