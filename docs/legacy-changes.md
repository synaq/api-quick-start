# Changes from the Legacy API

This section is for integrators upgrading from the pre-2019 SYNAQ API structure.

If you are building a new integration, this section does not apply to you.

---

## New product bundles

The legacy product suite has been replaced by a new bundle structure. Existing customers should be migrated to the new bundles. See [Migration Guide](migration-guide.md).

---

## Package creation — no edition codes required

In the legacy API, packages required explicit `product_group_code` and `editions` fields. These are no longer needed for new products — appropriate editions are assigned automatically. The fields are still accepted for legacy products.

---

## One package → one domain

All new products are single-domain packages. To manage multiple domains under the same product, create a separate package for each domain.

This change eliminates the need for the legacy partial cancellation workflow. To stop service on one domain in a multi-domain subscription, simply delete that domain.

---

## Partial cancellations

The partial cancellation workflow is deprecated and not documented here. The recommended approach is to migrate the domain to a product that excludes the unwanted service.

If you need the old workflow for maintenance purposes, consult the [Legacy API documentation](https://github.com/synaq/api-quick-start/tree/legacy).

---

## Unlinking domains

Unlinking a domain from a package as a way to remove services is deprecated. Use product migration instead.

---

## Immediate domain provisioning

The `provision_immediately` flag on domain creation is deprecated for new products — all current products require service field configuration before provisioning.

The `provision_immediately` flag **is still supported and available for mailboxes**.
