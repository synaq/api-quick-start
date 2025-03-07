# SYNAQ API Quick Start Guide

Valid for release 2022-04-29.2 and later of the SYNAQ API, last updated 2023-05-24.

# Introduction

The SYNAQ API allows resellers integrated with it to directly manipulate customer data, and provision and manage SYNAQ services under domains for their customers.

# Contents
- [New API and product structure](#new-api-and-product-structure)
  * [Legacy API documentation](#legacy-api-documentation)
- [Basic concepts](#basic-concepts)
  * [Products](#products)
  * [Editions](#editions)
  * [Organisational units (OUs)](#organisational-units-ous)
  * [Packages](#packages)
  * [Domains](#domains)
  * [Mailboxes](#mailboxes)
  * [Distribution Lists](#distribution-lists)
  * [Calendar Resources](#calendar-resources)
  * [Service fields](#service-fields)
  * [Asynchronous actions](#asynchronous-actions)
  * [Domain (and mailbox) actions](#domain-and-mailbox-actions)
  * [Updating active domains](#updating-active-domains)
  * [Migrating between products](#migrating-between-products)
  * [Moving domains between companies](#moving-domains-between-companies)
  * [Product bundles](#product-bundles)
- [Prerequisites](#prerequisites)
- [Online API documentation and development sandbox](#online-api-documentation-and-development-sandbox)
- [Developer support](#developer-support)
- [Advanced use cases](#advanced-use-cases)
- [Usage examples](#usage-examples)
  * [Request authentication](#request-authentication)
  * [OUs](#ous)
    + [Creating a company underneath an existing reseller](#creating-a-company-underneath-an-existing-reseller)
    + [Deleting an organisation](#deleting-an-organisation)
  * [Packages](#packages-1)
    + [Look up all possible package combinations under a given company](#look-up-all-possible-package-combinations-under-a-given-company)
    + [Creating a Package](#creating-a-package)
      - [Package codes](#package-codes)
    + [Provisioning a package](#provisioning-a-package)
    + [Deleting a package](#deleting-a-package)
  * [Domains](#domains-1)
    + [Create a new standalone domain which can be linked to an existing package](#create-a-new-standalone-domain-which-can-be-linked-to-an-existing-package)
    + [Linking an existing domain to a package](#linking-an-existing-domain-to-a-package)
    + [Creating a new domain and linking it to an existing package automatically](#creating-a-new-domain-and-linking-it-to-an-existing-package-automatically)
    + [Configuring the service fields on a domain](#configuring-the-service-fields-on-a-domain)
    + [Provisioning a domain](#provisioning-a-domain)
    + [Closing a domain](#closing-a-domain)
    + [Reactivating a domain](#reactivating-a-domain)
    + [Deleting a domain](#deleting-a-domain)
    + [Unlinking a domain from a package](#unlinking-a-domain-from-a-package)
    + [Refreshing a domain](#refreshing-a-domain)
  * [Mailboxes](#mailboxes-1)
    + [Searching for mailboxes](#searching-for-mailboxes)
    + [Creating a mailbox under a domain](#creating-a-mailbox-under-a-domain)
    + [Provisioning a mailbox](#provisioning-a-mailbox)
    + [Creating a mailbox under a domain and provisioning it immediately](#creating-a-mailbox-under-a-domain-and-provisioning-it-immediately)
    + [Updating a mailbox](#updating-a-mailbox)
    + [Suspending a mailbox](#suspending-a-mailbox)
    + [Closing a mailbox](#closing-a-mailbox)
    + [Reactivating a mailbox](#reactivating-a-mailbox)
    + [Unlocking a mailbox](#unlocking-a-mailbox)
    + [Generating an authentication token for a mailbox](#generating-an-authentication-token-for-a-mailbox)
    + [Changing the edition (class of service) of a mailbox](#changing-the-edition-class-of-service-of-a-mailbox)
    + [Checking DL membership for a mailbox](#checking-dl-membership-for-a-mailbox)
    + [Deleting a mailbox](#deleting-a-mailbox)
    + [Hashed passwords](#hashed-passwords)
  * [Distribution Lists](#distribution-lists-1)
    * [Searching for distribution lists](#searching-for-distribution-lists)
    * [Fields for distribution lists](#fields-for-distribution-lists)
    * [Creating a distribution list on a domain](#creating-a-distribution-list-on-a-domain)
    * [Provisioning a distribution list](#provisioning-a-distribution-list)
    * [Updating a distribution list](#updating-a-distribution-list)
    * [Deleting a distribution list](#deleting-a-distribution-list)
  * [Calendar Resources](#calendar-resources-1)
    * [Searching for calendar resources](#searching-for-calendar-resources)
    * [Fields for calendar resources](#fields-for-calendar-resources)
    * [Creating a calendar resource on a domain](#creating-a-calendar-resource-on-a-domain)
    * [Provisioning a calendar resource](#provisioning-a-calendar-resource)
    * [Updating a calendar resource](#updating-a-calendar-resource)
    * [Deleting a calendar resource](#deleting-a-calendar-resource)
  * [Usage reporting](#usage-reporting)
    + [Detailed usage reports](#detailed-usage-reports)
    + [Mailbox metadata](#mailbox-metadata)
- [Full workflow examples for advanced operations](#full-workflow-examples-for-advanced-operations)
  * [Polling an asynchronous action](#polling-an-asynchronous-action)
  * [Migrating a domain from one product to another](#migrating-a-domain-from-one-product-to-another)
    + [Migrating from Cloud Mail Legacy to Cloud Mail Standard with Branding](#migrating-from-cloud-mail-legacy-to-cloud-mail-standard-with-branding)
    + [Migrating from Cloud Mail Plus with Branding to Securemail Standard](#migrating-from-cloud-mail-plus-with-branding-to-securemail-standard)
  * [Checking if a domain can be migrated to a new product and how the migration will affect mailbox allocations](#checking-if-a-domain-can-be-migrated-to-a-new-product-and-how-the-migration-will-affect-mailbox-allocations)
    + [Checking if a domain with Securemail Standard can be migrated to Securemail Premium](#checking-if-a-domain-with-securemail-standard-can-be-migrated-to-securemail-premium)
    + [Checking if a domain with Securemail Standard can be migrated to Cloud Mail Standard](#checking-if-a-domain-with-securemail-standard-can-be-migrated-to-cloud-mail-standard)
    + [Checking if a domain with Cloud Mail Standard can be migrated to Securemail Standard with mailboxes still active](#checking-if-a-domain-with-cloud-mail-standard-can-be-migrated-to-securemail-standard-with-mailboxes-still-active)
    + [Checking if a domain with Cloud Mail Standard can be migrated to Securemail Standard with no active mailboxes remaining](#checking-if-a-domain-with-cloud-mail-standard-can-be-migrated-to-securemail-standard-with-no-active-mailboxes-remaining)
    + [Checking if a domain with legacy Cloud Mail can be migrated to Cloud Mail Plus](#checking-if-a-domain-with-legacy-cloud-mail-can-be-migrated-to-cloud-mail-plus)
  * [Moving a domain from one company to another](#moving-a-domain-from-one-company-to-another)
    + [Example](#example)
- [Changes between the legacy SYNAQ API and the current structure](#changes-between-the-legacy-synaq-api-and-the-current-structure)
  * [New product bundles](#new-product-bundles)
  * [Product group codes and edition codes](#product-group-codes-and-edition-codes)
  * [Single domain products](#single-domain-products)
  * [Partial cancellations](#partial-cancellations)
  * [Unlinking domains from packages](#unlinking-domains-from-packages)
  * [Immediate provisioning of domains](#immediate-provisioning-of-domains)

# New API and product structure

This document describes the new SYNAQ API and product structure, as implemented with release 20190814.01 on 2019-08-17. If you are still integrated to the legacy API, it is recommended that you upgrade to the new product structure as soon as possible. These documentation sections have critical details you will need to achieve this:

* [Product bundles](#product-bundles)
* [Migrating a domain from one product to another](#migrating-a-domain-from-one-product-to-another)
* [Changes between the legacy SYNAQ API and the current structure](#changes-between-the-legacy-synaq-api-and-the-current-structure)

## Legacy API documentation

If you are still integrated with the legacy API and need access to documentation for that structure for critical maintenance, please consult the legacy API documentation here: https://github.com/synaq/api-quick-start/tree/legacy

# Basic concepts

## Products

Products are combinations of physical services sold by SYNAQ. Examples of products include Cloud Mail, our cloud based email hosting service, Securemail, our email security service, etc. Products are purchased via the API by creating packages, as described below. A package is an instance of a given product.

## Editions

Products which offer the capability to provision and manipulate individual mailboxes, such as Cloud Mail, Mail Management and Continuity, can be configured with different classes or levels of service, to allow the package to be tailored to the end customer's specific requirements and budget. A package will be created with one or more editions, and where appropriate, mailboxes may be assigned to these editions.

**Compatibility note**
Unlike the legacy SYNAQ API, it is no longer necessary to assign editions to packages at the time of creating the package. Appropriate editions are now allocated automatically for all products.

## Organisational units (OUs)

An OU is the most basic structure which can be created by the API. OUs represent a reseller's customers, either as "sub-resellers", who can in turn sell to their own clients, or "companies", which are direct clients of the reseller.

The API allows creation of organisational units, under which packages and domains must then be created and linked to each other in order to provision services.

## Packages

A package is an instance of a given SYNAQ product under a particular organisational unit. For example, if a client wishes to buy the SYNAQ Securemail Standard product, an OU would be created to represent that client, and then a SYNAQ Securemail Standard package would be created under that OU.

## Domains

A domain is a representation in the API of an actual customer domain. Domains exist underneath OUs, and must be linked to one or more packages in order to be provisioned. The packages linked to a given domain will determine which actual services are provisioned for that domain on the SYNAQ platform.

## Mailboxes

For products where mailboxes can be managed directly via the API (all variants of Cloud Mail, Mail Management and Continuity), mailboxes in the API represent actual mailboxes on those services. The API allows creation of mailboxes underneath domains, and then allows the mailboxes to be provisioned onto the actual backing services on the SYNAQ platform.

## Distribution Lists
Distribution lists are special email addresses which deliver any mail they receive to all members of the list. For products where they can be configured (all variants of Cloud Mail), a distribution list record in the API mirrors and represents an unerlying distribution list on the platform.

The API allows creation and provisioning of distribution lists under domains, as well as updating and removal of those distribution lists.

**Note**
The current version of the API does not allow the creation or management of distribution lists with restricted sender grants. A request for that feature is in the queue for future consideration.

## Calendar Resources

Calendar resources are special email addresses which act as place holders for shared resources when booking meetings. A calendar resource could refer to a meeting room, a projector, or in general to any shared location or equipment which needs to be booked prior to use.

The API allows creation and provisioning of calendar resources under active domains on all Cloud Mail products, as well as deletion of calendar resources. Support for updating calendar resources will be added in an upcoming release.

## Service fields

All products require additional configuration for provisioning to be possible. This includes information like the delivery destinations, authentication details, and credentials for admin users.

Service fields are managed via the API through a PATCH call made on a domain object. This must be done before attempting to provision the domain though a provisioning action.

## Asynchronous actions

The SYNAQ API abstracts a message queue based services bus used on the SYNAQ platform to queue and complete actual provisioning tasks on the services on our platform. As some tasks may require a short time to complete, or wait in the queue at particularly busy times, all of these services bus tasks are represented in the API by asynchronous actions.

Asynchronous actions will generally complete very quickly, but can be delayed in especially busy situations, or for services where some manual intervention might be required on the SYNAQ platform. In general, all asynchronous actions on products other than SYNAQ Archive should complete within a minute or less, and almost always complete within a few seconds.

When asynchronous actions are created, the API returns their location to the client in an HTTP header. This location can be used to poll the action and determine its state. When a problem occurs, the action will also report the reason for the failure, which can be relayed to SYNAQ support for assistance.

Full documentation on the use of asynchronous actions may be found in the section on [Polling an asynchronous action](#polling-an-asynchronous-action)

## Domain (and mailbox) actions

The most important asynchronous actions in the API are actions on domains and mailboxes. These can be triggered explicitly by creating them on the domain or mailbox actions endpoint in the API, and are sometimes triggered implicitly by certain endpoints, or by specific configuration settings on some endpoints. In cases where this may happen, this document will highlight that possibility.

The most important and frequently used domain and mailbox action is the `Provision` action. This initiates the steps needed to actually provision an already configured domain (or mailbox) on SYNAQ's platform.

For detailed instructions on how to deal with asynchronous actions, see [Polling an asynchronous action](#polling-an-asynchronous-action)

## Updating active domains

The API supports updating of the service data of active domains. This allows domain-specific service information which was set at provisioning time to be updated on the existing domain, for example, changing the delivery destination, the password for SMTP authentication, or IP addresses for IP based authentication, on a Securemail domain, or similar authentication and/or routing information for an existing Branding domain.

The procedure for performing these actions is explained in detail in the addenda to the documentation section on [Configuring the service fields on a domain](#configuring-the-service-fields-on-a-domain)

## Migrating between products

The API supports migrating domains between products by linking a domain to an appropriately configured new package. This allows resellers to easily up-sell end customers to product bundles with new options, or to move them to products which do not include services which they do not need, rather than having to cancel an account entirely.

The procedure for migrating between products is explained in detail in the section on [Migrating a domain from one product to another](#migrating-a-domain-from-one-product-to-another)

## Moving domains between companies

The API supports moving domains between any two companies to which the user has access. This is done by configuring a new package on the company to which the domain should move, with special configuration to indicate that a domain move should take place, and linking the domain to that new package.

The procedure for moving domains between companies explained in detail in the section on [Moving a domain from one company to another](#moving-a-domain-from-one-company-to-another)

## Product bundles

The current API structure implements saleable products as bundles. These include several underlying services on the SYNAQ platform. Here is a very brief list of the products and which services they include. For detailed information and sales collateral, please contact your SYNAQ account manager.

The product and edition codes needed to set up these products are listed in the section on [Creating a Package](#creating-a-package)

* Cloud Mail Lite
  * 2GB Zimbra mailboxes with POP and IMAP support
  * SYNAQ Securemail with Identity Threat Protection
* Cloud Mail Standard
  * 50GB / 100GB Zimbra mailboxes with support for POP, IMAP, ActiveSync and optional Zimbra archiving
  * Optional 2GB Zimbra mailboxes with POP and IMAP support
  * SYNAQ Securemail with Identity Threat Protection
* Cloud Mail Plus
  - 50GB / 100GB Zimbra mailboxes with support for POP, IMAP, ActiveSync, MAPI and optional Zimbra archiving
  - Optional 2GB Zimbra mailboxes with POP and IMAP support
  - SYNAQ Securemail with Identity Threat Protection
* Cloud Mail (Lite / Standard / Plus) with Branding
  The same as the above product, but also including SYNAQ Branding
* Securemail Standard
  * SYNAQ Securemail with Identity Threat Protection
  * SYNAQ Branding
* Securemail Premium
  - SYNAQ Securemail with Identity Threat Protection
  - LinkShield protection
  - Data leak prevention
  - SYNAQ Branding
* Branding Standard
  * SYNAQ Branding
  * SYNAQ Securemail with Identity Threat Protection
* SecureArchive
  * SYNAQ Archive with 10 year retention
  * Managed Securemail (no UI access)
* Continuity
  * SYNAQ Continuity
  * Managed Securemail (no UI access)
* Mail Management Standard
  * SYNAQ Continuity
  * SYNAQ Securemail with Identity Threat Protection
  * SYNAQ Branding
  * SYNAQ Archive with 10 year retention
* Mail Management Premium
  - SYNAQ Continuity
  - SYNAQ Securemail with Identity Threat Protection
  - SYNAQ Branding
  - SYNAQ Archive with unlimited retention

# Prerequisites

To use the API, an integrator requires the top level GUID assigned to their reseller, and a valid API key. These will be provided by SYNAQ upon request. For new resellers, we will generally only allocate credentials for our staging environment, enabling the integrator to develop and test their integration client without the possibility of affecting production services or incurring billing.

Once the integrator is satisfied that their integration is stable and ready to serve their customers, we will provide access credentials for the production API.

# Online API documentation and development sandbox

SYNAQ supplies an online API documentation and sandbox tool. The tool lists all possible calls to the SYNAQ API, together with their expected payloads, and possible responses.

By clicking on the `Sandbox` tab under each documented endpoint, the user may also make a live request to the API for that call directly from the tool. This was intended to make the process of gaining sample payloads and responses for the API easier, and also to aid in troubleshooting.

The tool is available on our staging API infrastructure at this URL:

<https://sta-q.synaq.com/api-doc>

Please log in using your user interface credentials, not your API key. This will be provided along with other staging configuration information.

# Developer support

The SYNAQ development team is available to support developers with integration partners via email. Please contact the team directly on <dev@synaq.com> for assistance.

# Advanced use cases

The API supports advanced use cases not covered in this guide. These are normally either for use by SYNAQ internally, or were developed for specific integrators. They are not covered here for the sake of brevity, but are available to all integrators on request. Please contact SYNAQ for more information on these use cases.

# Usage examples

## Request authentication

All requests to the SYNAQ API must be authenticated using SWSS authentication, injected into the HTTP headers. These headers are required in all requests:

```
synaq-api-user: {reseller API user name}
synaq-api-salt: {current UNIX timestamp}
synaq-api-hash: {user name, API key and salt, concatenated and hashed in SHA1}
```

The user name and API key are provided by SYNAQ. The salt is a standard UNIX timestamp**, and the hash is calculated by concatenating the username, API key and salt, and hashing them using the SHA1 hashing algorithm.

** While a standard timestamp will suffice, it is important that timestamps not be reused, to prevent replay attacks. If you expect your API client to make more than one request per second, we advise using a modified timestamp which includes milliseconds or microseconds to increase granularity.

**PHP Hash Generation Example**

```
$username = 'client@client.com';
$apiKey = 'some-long-api-key-string-provided-by-us';
$salt = microtime(true) * 10000;
$hash = sha1($username . $apiKey . $salt);
```

## OUs

### Creating a company underneath an existing reseller

Create a company titled "Acme Ltd", belonging to the authenticated reseller.

```
POST /api/v1/ous/{reseller-guid}/companies.json
```

**reseller-guid:** The root level reseller GUID provided by SYNAQ, or a sub-reseller GUID, which can be obtained by first creating a sub-reseller** via the API.

**Request payload:**

```
{
  "ou": {
    "title": "Acme Ltd",
    "client_ref": "al",
    "phone_number": "0113216547",
    "vat_numer": "987654320",
    "physical_address": {
      "line_1": "20 Long Street",
      "city": "Johannesburg",
      "postal_code": "4321",
      "country": "ZA"
    }
  }
}
```

The `title` field on the OU, and `line_1`, `city`, `postal_code` and `country` on the `physical_address` object are all required fields. All other fields are optional.

**Response headers:**

```
201 Created
location: /api/v1/ous/{ou-guid}
```

Visiting the location will show the basic OU information. The OU GUID can also be extracted from the `location` header.

** To create a sub-reseller, follow the same process as creating as for company creation, but target the appropriate endpoint:

```
POST /api/v1/ous/{reseller-guid}/subresellers.json
```

### Deleting an organisation

If a company has no active packages associated with it, its records may be deleted form the API using a delete endpoint.

```
DELETE /api/v1/ous/{ou-guid}.json
```

**Response headers:**

```
204 No Content
```

## Packages

### Look up all possible package combinations under a given company

```
OPTIONS /api/v1/ous/{company-guid}/packages.json
```

The response will show all possible product and edition combinations for new packages under the given company. Due to this response being quite detailed and lengthy, it has been omitted from this document. Please use the API sandbox tool to generate a sample response for your own test company.

This response can be used to dynamically generate forms with available products on your client system.

### Creating a Package

```
POST /api/v1/ous/{company-guid}/packages.json
```

**Request payload:**

```
{
  "package": {
    "code": "{package-code}"
  }
}
```

**Response headers:**

```
201 Created
location: /api/v1/packages/{package-guid}
```

#### Package codes

* **CLM-LT** Cloud Mail Lite
* **CLM-STD** Cloud Mail Standard
* **CLM-PLUS** Cloud Mail Plus
* **CLM-LTB** Cloud Mail Lite with Branding
* **CLM-STD-B** Cloud Mail Standard with Branding
* **CLM-PLUS-B** Cloud Mail Plus with Branding
* **SM-STD** Securemail Standard
* **SM-PREM** Securemail Premium
* **BRD-STD** Branding Standard
* **AR-SEC** SecureArchive
* **CT-STD** Continuity
* **MM-STD** Mail Management Standard
* **MM-PREM** Mail Management Premium

### Provisioning a package

In cases when a new package is being added to an existing domain, or the domain has migrated to a new package, the API permits a provision action to be created on a package rather than a domain. The process for doing this is the same as for provisioning actions created directly on domains, but using the package actions endpoints.

**Note:** See the domain documentation below for more information on linking packages and domains.

```
POST /api/v1/packages/{package-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Provision"
	}
}
```

**Response headers:**

```
202 Accepted
location: /api/v1/packages/{package-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Deleting a package

If the domain linked to the package is in the `deleted` state, or if the package has no domain record linked to it, the package itself may be deleted by a `DELETE` endpoint.

```
DELETE /api/v1/packages/{package-guid}.json
```

**Response headers:**

```
204 No Content
```

## Domains 

### Create a new standalone domain which can be linked to an existing package

```
POST /api/v1/ous/{company-guid}/domains.json
```

**Request payload:**

```
{
  "domain": {
    "domain_name": "acme.com"
  }
}
```

**Response headers:**

```
201 Created
location: /api/v1/domains/{domain-guid}
```

### Linking an existing domain to a package

Before packages or domains can be provisioned, or have their service fields configured for products where that is required, they must be linked in the API.

```
LINK /api/v1/packages/{package-guid}/domains/{domain-guid}.json
```

**Response headers:**

```
204 No Content
```

### Creating a new domain and linking it to an existing package automatically

To simplify the process of creating domains and linking them, the API provides a convenience method where a domain is created underneath a package.

In reality, the domain is still owned by the OU which owns the package, but using this method, the domain is linked to the package immediately, negating the need for the intermediary linking step.

```
POST /api/v1/packages/{package-guid}/domains.json
```

**Request payload:**

```
{
  "domain": {
    "domain_name": "acme.com"
  }
}
```

**Response headers:**

```
201 Created
location: /api/v1/domains/{domain-guid}
```

### Configuring the service fields on a domain

Additional service information is required to be configured before a domain can be provisioned. Which information is required is determined by the package(s) linked to the domain, so linking must be completed before these steps.

To determine which fields require information, the general use GET endpoint for domains may be used with the given domain GUID, and the `fields` object may be inspected.

**Important Note:** For legacy compatibility reasons, any endpoints which expose the **smtp_username** and **smtp_password** fields should have the same values duplicated as **username** and **password**.

**Note:** The endpoint also exposes a legacy `service_fields` object, which was used in a previous mechanism for retrieving information passed back from the services. Please do no confuse that object for `fields` as described above.

```
GET /api/v1/domains/{domain-guid}.json
```

**Sample Response for a Securemail Standard domain (abbreviated):**

```
{
  "guid": "domain-guid",
  "domain_name": "some-domain.com",
  "fields": {
    "mail_delivery_location": null,
    "mail_delivery_alternative": null,
    "auth_method": "smtp",
    "username": null,
    "password": null,
    "admin_username": null,
    "admin_password": null,
    "admin_first_name": null,
    "admin_last_name": null,
    "service_provider": "other"
  }
}
```

This shows required fields for the selected package combination, and their current values, if any. Once the fields are known, or using a predefined set of fields for given products in the client, the service information should be requested from the end user, and then updated on the API using the special PATCH endpoint for service fields on the given domain


```
PATCH /api/v1/domains/{domain-guid}/servicefields.json
```

**Request payload:**

```
{
	"fields": {
   		"mail_delivery_location": "smtp.some-domain.com",
   		"mail_delivery_alternative": "smtp2.some-domain.com",
   		"auth_method": "smtp",
   		"username": "smtp-username",
   		"password": "smtpPassw0rd!"
   		"admin_username": "jane.doe@customer.com",
    	"admin_password": "somePass1234%$",
    	"admin_first_name": "Jane",
    	"admin_last_name": "Doe"
   	}
}
```

**Response headers:**

```
204 No Content
```

**Important**

At this point, if the domain was already active, the will have entered the `stale` state, and will need to be refreshed.

Please see the documentation section on [Refreshing a domain](#refreshing-a-domain) for detailed instructions.

### Provisioning a domain

Once service configuration information has been populated in the service fields, the domain may be provisioned using an asynchronous `Provision` action as described in the basic concepts section above.

```
POST /api/v1/domains/{domain-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Provision"
	}
}
```

**Response headers:**

```
202 Accepted
location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

The `location` header references the location of the newly created provisioning action. That location may be polled by the client to retrieve the state of the action. Actions should be polled until they either return a state of `finished` or `error`. In the case of an `error` state, an accompanying array of error messages will also be returned as the `errors` field to explain why provisioning did not succeed.

```
GET /api/v1/domains/{domain-guid}/actions/{action-id}
```

**Sample response:**

```
{
  "id": 1234,
  "action": "provision",
  "state": "finished",
  "errors": [],
  "exclusions": [],
}
```

The polling process for actions on packages is similar to that shown for domains above.

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Closing a domain

In some cases, an integrator may wish to temporarily suspend the operation of a domain. This is usually needed by integrators who want to enforce credit control on end customers by temporarily denying service, without completely tearing down the provisioned services.

A closed domain will not process incoming mail, nor allow outgoing mail to be relayed.

To close a domain, create an asynchronous `Close` action on the domain actions endpoint. Poll the action as usual.

```
POST /api/v1/domains/{domain-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Close"
	}
}
```

**Response headers:**

```
202 Accepted
location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Reactivating a domain

A closed domain may be reactivated (returned to service) by using the asynchronous `Activate` action.

```
POST /api/v1/domains/{domain-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Activate"
	}
}
```

**Response headers:**

```
202 Accepted
location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Deleting a domain

If an end customer has cancelled service for a domain, and an integrator wishes to remove it from service, this may be achieved by first creating an asynchronous `Delete` action for the domain, and then using the `DELETE` endpoint on the domain itself to remove all records of it.

```
POST /api/v1/domains/{domain-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Delete"
	}
}
```

**Response headers:**

```
202 Accepted
location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

Once the action reports a state of `finished`, the domain will be in the `deleted` state, and thus eligible for final deletion from the API records using the `DELETE` endpoint.

```
DELETE /api/v1/domains/{domain-guid}.json
```

**Response headers:**

```
204 No Content
```

**Notes:**

* The `DELETE` action is only permitted for domains in the `inactive` or `deleted` states. A domain in any other state can not be deleted from the API's records.

* For products which expose mailboxes directly as API objects, such as Cloud Mail, Continuity and Mail Management, a domain may not be deleted if it still contains mailboxes in provisioned states. 

  For this reason, the API will reject an asynchronous `Delete` action on any domain where provisioned mailboxes are still present. To delete such a domain, the mailboxes must be deleted first. Please refer to the mailbox management section below.

### Unlinking a domain from a package

Domains can also be unlinked from packages to which they are already linked. 

```
UNLINK /api/v1/packages/{package-guid}/domains/{domain-guid}.json
```

**Response headers for inactive domains:**

```
204 No Content
```

**Compatibility note:**

In the legacy API structure, it was possible to remove certain services a client no longer needs by unlinking a package from an active domain. While that workflow is still supported for backwards compatibility, it is deprecated and should not be used, so documentation for it has been omitted.

The recommended way to remove unwanted services from a domain in the new structure is to migrate the domain to a product bundle which does not include the unwanted service. Please see [Migrating a domain from one product to another](#migrating-a-domain-from-one-product-to-another) for details.

If you need to perform the old workflow for maintenance reasons, please consult the [Legacy API documentation](#legacy-api-documentation)

### Refreshing a domain

If a domain has entered the `state` state, some of its services need to be refreshed. This generally only occurs in the current version of the API after updating an active domain's service fields (see [Configuring the service fields on a domain](#configuring-the-service-fields-on-a-domain))

```
POST /api/v1/domains/{domain-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Refresh"
	}
}
```

**Response headers:**

```
202 Accepted
location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

Once the action reports a state of `finished`, the domain will be in the `active` state, and no further action is required.

## Mailboxes

### Searching for mailboxes

Any mailbox in the user's scope can be found using the global mailboxes endpoint. There are rich filters available on the call to limit the scope of mailbox searches further, most importantly, `domain_guid` and `domain_name`.

```
GET /api/v1/mailboxes.json?domain_guid={some_guid}&exact_match=1
```

```
{
  "page": 1,
  "limit": 5,
  "pages": 1,
  "total": 1,
  "_embedded": {
    "mailboxes": [
      {
        "guid": "SOME-GUID",
        "first_name": null,
        "last_name": "Last Name",
        "display_name": "Some Display Name"
        "from_name": null,
        "admin": false,
        "description": null,
        "branch": null,
        "hide_in_gal": false,
        "state": "active",
        "type": "mailbox",
        "packages": [
          {
            "state": "active",
            "code": "CLM-STD",
            "edition_code": "CLM-STD-50-ARCH",
            "guid": "SOME-PACKAGE-GUID"
          }
        ],
        "email": "address@domain.com",
        "aliases": [],
        "catch_all_address": null,
        "zimbra_mail_store": "some-host.domain.com",
        "account_locked": false,
        "auth_locked": false,
        "spam_locked": false
      }
    ]
```

The `exact_match` modifier returns only mailboxes which exactly match the filter criteria, If not set, any mailbox where the local part and domain name specified **are part of the local part and domain** will be returned.

In general, we encourage always using ```exact_match```, unless you have a specific use case requiring partial matches.

When `domain_guid` or `domain_name` are specified, the results will automatically be ordered by the local part of the address in ascending order. If descending order is required, set the `sort_descending` flag.

For a ful and up to date list of filters and modifiers, as well as pagination options, please see the API documentation tool.

### Creating a mailbox under a domain

For products where mailboxes are directly exposed via the API, such as Cloud Mail, Continuity and Mailbox Management, a mailbox may be created and provisioned under a domain once the domain has been provisioned.

**Notes:**

* Only the `email_local`, `password` and `last_name` fields are required. Many other fields are available. Please see the API documentation for a full description of all available fields.

* Clear text passwords provided in the `password` field are immediately hashed and stored as SSHA256 salted hashes in SYNAQ's backend systems.

* Since version 6.4 of the API (2018-09-21), you may omit the `password` field, and instead supply the `ssha_password` field, which accepts SSHA256 or SSHA salted and hashed passwords.

  Please see the section on [hashed passwords](#hashed-passwords) for more details.

* A mailbox edition may be specified using the optional `package_edition_code`. The code specified must be a valid edition on the package. If no code is specified, the default edition on the package, usually the simplest mailbox type, will be used. Specifying the edition this way is recommended, as it simplifies the provisioning process. The API provides a PATCH endpoint to allow the updating of mailbox editions at any time (also for active mailboxes). Please see examples related to changing a mailbox edition below, and the online sandbox tool, for more information.

```
POST /api/v1/domains/{domain-guid}/mailboxes.json
```

**Request payload:**

```
{
	"mailbox": {
		"email_local": "sample",
		"password": "Sample123$",
		"last_name": "Sample",
		"package_edition_code": "CLM-STD-100"
	}
}
```

**Response headers:**

```
201 Created
Location: /api/v1/mailboxes/{mailbox-guid}
```

### Provisioning a mailbox

Once created under a domain, a mailbox must be provisioned using an asynchronous `Provision` action before it will be available.

The action follows the standard process for asynchronous actions.

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Provision"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Creating a mailbox under a domain and provisioning it immediately

As a convenience method to simplify the mailbox provisioning workflow, the mailbox creation endpoint also offers the `provision_immediately` flag, as per the domain provisioning workflow. In this case, the API will likewise return a 202 Accepted and a location header referring to the automatically created provisioning action. The GUID of the newly created mailbox may be inferred by parsing the action location.

```
POST /api/v1/domains/{domain-guid}/mailboxes.json
```

**Request payload:**

```
{
	"mailbox": {
		"email_local": "sample",
		"password": "Sample123$",
		"last_name": "Sample",
		"package_edition_code": "CLM-STD-50-ARCH",
		"provision_immediately": true
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Updating a mailbox

Occasionally, it may be necessary to update the details of a mailbox, such as display name, the mailbox password, forwarding address, etc.

This can be done on inactive and provisioned mailboxes using a PATCH call on the mailbox endpoint. Only the fields to be changed should be included in the payload.

**Note:** When updating a provisioned mailbox, the API will automatically create an asynchronous `Update` action to propagate the change to actual services. The response will include a reference to the location of this action.

```
PATCH /api/v1/mailboxes/{mailbox-guid}.json
```

**Possible request payload:**

```
{
	"mailbox": {
		"password": "someNewPassw0rd!"
	}
}
```

**Response headers for inactive mailboxes:**

```
204 No Content
```

**Response headers for active mailboxes:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Suspending a mailbox

As a credit control measure, or to curb abusive behaviour, service to a single mailbox may be partially suspended via the API. A suspended mailbox will still receive mail destined for it, but will not be accessible to the end user.

This is done by creating an asynchronous `Suspend` action on the mailbox.

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Suspend"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Closing a mailbox

Closing a mailbox is a more severe form of service suspension. A closed mailbox will not be accessible to its end user, and mail delivery destined for the mailbox will also be rejected, resulting in mail destined for a closed mailbox bouncing. The existing contents of the mailbox are preserved. This may be employed as a more severe credit control or abuse mitigation measure, and also as a "soft delete" in cases where an end user has cancelled service, but the integrator wishes to preserve the data in their mailbox temporarily before finally deleting it.

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Close"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Reactivating a mailbox

A mailbox may be returned from either the suspended or closed states by creating an asynchronous `Activate` action.

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Activate"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Unlocking a mailbox

In certain cases, SYNAQ may temporarily restrict access to a mailbox as an emergency measure to combat abuse being committed using the mailbox. In these cases, the mailbox will show the `suspended` state on normal inspection, but also expose specific fields indicating the reason for the lock.

A locked mailbox cannot be revived using the activate action. To unlock the mailbox, the specific `Unlock` asynchronous action must be employed. It is also highly recommended that the mailbox password be changed, to prevent further abuse. Failing to change the password will result in a compromised account being locked again.

The state of the mailbox may be inspected using the normal mailbox GET endpoint.

```
GET /api/v1/mailboxes/{mailbox-guid}.json
```

**Sample response for a locked mailbox (abbreviated)**

```
{
	"guid": "mailbox-guid",
	"email":"mailbox@some-domain.com",
	"state": "suspended",
	"account_locked": true,
	"lock_type": "full",
	"lock_reason_code": "ABC123",
	"lock_reason": "A human readable reason for the mailbox lockout"
}
```

The simplest way to confirm a mailbox lock is if the `account_locked` flag is raised. The contents of the `lock_reason` field may be presented to support engineers by the user interface in order to allow them to diagnose the reason for the lockout before attempting to repair it.

If the issue is corrected, the mailbox may be unlocked using the asynchronous `Unlock` action.

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Unlock"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Generating an authentication token for a mailbox

Administrators or support staff may occasionally need to directly log in to a mailbox in order to trouble shoot problems with the web mail service, or assist clients in configuration or maintenance tasks. In these cases, it is undesirable to expose authentication credentials for the mailbox directly to the staff member, and they should instead be permitted to log in to the mailbox without a password, if that mailbox is in their scope.

To facilitate logins without exposing credentials to staff members, an authentication token can be generated for the mailbox, and used to directly authenticate the user to the web mail service, by redirecting the user's browser to the service and including the token as a request parameter.

**Important note:**

Integrators are strongly encouraged to restrict the availability of this use case to special trusted classes of users, and keep an audit log of its use, to prevent unauthorised use of the feature and associated security compromises.

**Generating tokens:**

Before a token can be retrieved for a mailbox, one must be generated using the `Token` action on the mailbox. This action is only permitted for active mailboxes.

**Request:**

```
POST /api/v1/mailboxes/{mailbox-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Token"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

**Retrieving tokens:**

Once a token has been generated for a mailbox, it can be retrieved using the special token endpoint on that mailbox. If no valid token exists, the endpoint will return an empty response. In this case, a new token must be generated. If an existing token is available and still within its validity period, it will be returned, along with the expiration date, so that the integrator can cache the date if desired.

Storing of the actual token itself is strongly discouraged, instead, it is recommended that only the expiry date of the most recent token is stored, and this is used to decide if a new token should be requested, or the existing token retrieved from the API without attempting to generate one first. Integrators should always anticipate the possibility that no valid token exists, and fall back to requesting one in that event.

Attempting to retrieve a token for a mailbox which is not in the `active` state will result in an error.

**Request:**

```
GET /api/v1/mailboxes/{mailbox-guid}/token.json
```

**Response if a valid token exists:**

```
200 OK

{
	"authentication_token": "EXAMPLE_TOKEN",
	"authentication_token_expiry": "2020-10-10T10:10:00+0200"
}
```

**Response if no valid token is available:**

```
204 No Content
```

**Directing the user's browser to the web mail service:**

The user can new be directed to the mailbox using the token and the public web mail host name. Having the browser perform this operation in a new tab or new window is highly recommended. The URL to direct the browser to is as follows:

```
https://{domain.public.service.host.name}/service/preauth?authtoken={token}&isredirect=1&adminPreAuth=1
```

For example, if the hostname were `mail.provider.com` and token string `SOME_TOKEN_1234`, the browser should be directed to:

```
https://mail.provider.com/?zauthtoken=SOME_TOKEN_1234
```

Since tokens can remain valid for extended periods depending on server configuration, they should never be transmitted across unencrypted connections, so integrators are strongly encouraged to strictly enforce the use of HTTPS for public service host names. Integrators are also encouraged to make the host name configurable, to allow for white labelling or custom configurations for individual customers if desired.

### Changing the edition (class of service) of a mailbox

From time to time, it may be necessary to change the package edition associated with a mailbox. This has the effect of also changing the actual class of service of a provisioned mailbox.

This may be used to upgrade a mailbox, especially in cases where an integrator wishes to up-sell users onto a more premium package, or to downgrade a user, in cases where end customers might want to manage their spend by rationalising the service classes offered to their internal users.

If the mailbox is already provisioned, this call will automatically create an asynchronous update action. The change to the actual service will be visible when that action is completed.

```
PATCH /api/v1/mailboxes/{mailbox-guid}/packages/{package-guid}/edition.json
```

**Request payload:**

```
{
	"edition": {
		"code": "CLM-STD-100-ARCH"
	}
}
```

**Response headers for inactive mailboxes:**

```
204 No Content
```

**Response headers for active mailboxes:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

**Note:** Any valid edition code on the package may be specified. In the above example, a mailbox is upgraded to the 100GB option with Archiving on a Cloud Mail Standard package. It could also have been downgraded to a basic 2GB mailbox by using the appropriate edition code, for example `CLM-LT`. The valid editions on a package may be verified by using the GET call on the package itself.

### Checking DL membership for a mailbox

For mailboxes on any of the Cloud Mail products, it is possible to determine which distribution lists (if any) the mailbox belongs to, as of release 2022-02-18.1.

**Note**:

* Only avilable for active (provisined) mailboxes.
* Only distribution lists belonging to the same company will be returned.

```
GET /api/v1/mailboxes/{mailbox-guid}/membership.json
```

**Sample response**

```
{
  "guid": "mailbox-guid",
  "email": "mailbox@domain.com",
  "dls": [
    {
      "guid": "dl-guid",
      "hide_in_gal": false,
      "state": "active",
      "type": "dl",
      "email": "some.dl@domain.com",
      "catch_all_address": false,
    },
    {
      "guid": "another-dl-guid",
      "hide_in_gal": false,
      "state": "active",
      "type": "dl",
      "email": "another.dl@domain.com",
      "catch_all_address": false,
    }
  ]
}
```

### Deleting a mailbox

If a mailbox is no longer in use and the integrator wishes to remove it entirely from services, the mailbox may be deleted using the `DELETE` call in the API. Take note that this automatically creates an asynchronous `Delete` action, which may be polled until the deletion is complete.

Deleting a mailbox permanently removes all of its associated data from services. There is no possibility to undo this action.

To simplify the process, it is not necessary to separately create a `Delete` action. This is handled automatically by the `DELETE` endpoint. For active mailboxes, a reference to the delete action will be returned.

```
DELETE /api/v1/mailboxes/{mailbox-guid}.json
```

**Response headers for inactive mailboxes:**

```
204 No Content
```

**Response headers for active mailboxes:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

**Note:** Unlike other asynchronous actions on API objects. The delete action will never present a `finished` state. This is because the one-step mailbox removal process also deletes the actual mailbox record in the case of a successful delete action, deleting the action along with it. Thus, the action may be polled while it is in progress, but when finished, polling it will return a `404 Not Found`, as will any attempt to retrieve mailbox records.

So, for this special use case, the deletion may be assumed to be complete as soon as a poll to the action or the mailbox itself returns not found.

### Hashed passwords

Since version 6.4 of the SYNAQ API (released on 2018-09-21), all endpoints which accept the clear text `password` field on mailboxes (`POST` for creating, `PATCH` for updating) now also accept a new `ssha_password` field, which may be provided instead of the clear text `password` field.

This allows API clients to provision or update mailbox passwords, without ever exposing the original clear text password to the SYNAQ API or any of our backing services.

Supported hashes are salted SHA256 or salted SHA1. SHA256 is recommended. The salt should be four securely generated random bytes, and must be appended to the end of the hash in order to allow our backing services to authenticate user passwords against it.

The hashing scheme used should be indicated by a format marker at the beginning of the string, either `{SSHA256}` or `{SSHA}`

Some pseudocode examples for computing the hashes and packing them as suitable strings for the `ssha_password` field are provided below.

**Example algorithm for salted SHA256**

```
salt = four_secure_random_bytes()
ssha_password = "{SSHA256}" + base64 ( sha256( password + salt) + salt )
```

**Example algorithm for salted SHA1**

```
salt = four_secure_random_bytes()
ssha_password = "{SSHA}" + base64 ( sha1( password + salt) + salt )
```

## Distribution Lists

Distribution lists are a special class of email address which deliver to multiple recipients (members). The API supports configuration and management of distribution lists for Cloud Mail.

### Searching for distribution lists

The API exposes a global distribution list endpoint. The operation is the same as the global mailbox endpoint, please see [Searching for mailboxes](#searching-for-mailboxes) for more details, or consult the API documentation tool.

```
GET /api/v1/dls.json?domain_guid={SOME-GUID}&exact_match=1
```

### Fields for distribution lists

The important fields for implementing distribution lists are listed here, a full list is available via the API documentation tool. For a successful implementation of DLs, all of these fields should be supported. They can be specified both when creating and updating DLs:

* `email_local` : The local part of the email address for the distribution list under the domain.

* `hide_in_gal`: Boolean flag indicating if the distribution list should be hidden from the global address list

* `members`: An array of objects describing members of the distribution list, taking the following form:

  ```
  			[
  				"local": "local.part",
  				"domain": "domain.name"
  			]
  ```

**Notes**

Specifying a different `email_local` when updating the DL wil result in its being renamed. When performing updates, fields which should not change should be omitted from the payload.

The `catch_all_address` field which is visible in the interactive documentation tool for new DLs is deprecated and will be removed in a future version of the API. It should not be implemented for new implementations of DLs in the API, and will not be supported by SYNAQ.

### Creating a distribution list on a domain

```
POST /api/v1/domains/{domain-guid}/dls.json
```

**Request payload**
```
{
	"dl": [
		"email_local": "some.group",
		"members": [
			[
				"local": "some.user",
				"domain": "some-domain.com"
			],
			[
				"local": "some.other.user",
				"domain": "some-other-domain.com"
			]
		]
	]
}
```

**Response headers:**

```
201 Created
Location: /api/v1/dls/{dl-guid}
```

**Note**

From the above example, it should be obvious that the members of a distribution list do not have to be on the same domain as the distribution list itself.

### Provisioning a distribution list

Once created under a domain, a distribution list must be provisioned using an asynchronous `Provision` action before it will be available.

The action follows the standard process for asynchronous actions.

```
POST /api/v1/dls/{dl-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Provision"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/dls/{dl-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Updating a distribution list

Both inactive and provisioned DLs can be updated using a PATCH call on the DL endpoint. Only the fields to be changed should be included in the payload.

**Note:** When updating a provisioned DL, the API will automatically create an asynchronous `Update` action to propagate the change to actual services. The response will include a reference to the location of this action.

```
PATCH /api/v1/dls/{dl-guid}.json
```

**Possible request payload:**

```
{
	"dl": {
		"members": [
			[
				"local": "some.user",
				"domain": "some-domain.com"
			],
			[
				"local": "some.new.user",
				"domain": "some-other-domain.com"
			]
		]
	}
}
```

**Response headers for inactive DLs:**

```
204 No Content
```

**Response headers for active DLs:**

```
202 Accepted
Location: /api/v1/dls/{dl-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Deleting a distribution list

Unlike mailboxes, active distribution lists must be "deleted" from the platform in the same way domains are deleted, by posting an asynchronous `Delete` action for the DL.

**Deletion of an active DL from the platform**

```
POST /api/v1/dls/{dl-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Delete"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/dls/{dl-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

Once the action completes, the inactive DL will be automatically deleted from the API records.

**Deleting an inactive DL from the API records**

If a distribution list was never provisioned, the record can be deleted from the API directly.

```
DELETE /api/v1/dls/{dl-guid}.json
```

**Response headers:**

```
204 No Content
```

## Calendar Resources

Calendar resources are a special class of email address which acts as a place holder for shared locations or equipment which must be booked for meetings or other uses.

### Searching for calendar resources

The API exposes a global calendar resource endpoint. The operation is the same as the global mailbox endpoint, please see [Searching for mailboxes](#searching-for-mailboxes) for more details, or consult the API documentation tool.

```
GET /api/v1/resources.json?domain_guid={SOME-GUID}&exact_match=1
```

### Fields for calendar resources

The important fields for implementing calendar resources are listed here, a full list is available via the API documentation tool. For a successful implementation of calendar resources, all of these fields should be supported:

* `email_local` : The local part of the email address for the calendar resource under the domain.
* `password`: A plain text password for the calendar resource. This will be automatically converted to an SSHA266 hash by the API if provided. It can be omitted if the `ssha_password` field is provided.
* `ssha_password`: An SSHA256 hashed password, marked with a {SSHA256} format tag. This is a more secure alternative to sending a plain text password.
* `display_name`: The display name to use for the calendar resource in the global address list and its responses to meeting requests.
* `resource_type`: The type of calendar resource, valid values are `Location` and `Equipment`.

### Creating a calendar resource on a domain

```
POST /api/v1/domains/{domain-guid}/resources.json
```

**Request payload**

```
{
	"resource": [
		"email_local": "some.boardroom",
		"password": "SomePassw0rd$",
		"display_name": "Some boardroom",
		"resource_type": "Location"
	]
}
```

**Response headers:**

```
201 Created
Location: /api/v1/resources/{resource-guid}
```

### Provisioning a calendar resource

Once created under a domain, a calendar resource must be provisioned using an asynchronous `Provision` action before it will be available.

The action follows the standard process for asynchronous actions.

```
POST /api/v1/resources/{resource-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Provision"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/resources/{resource-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Updating a calendar resource

Both inactive and provisioned calendar resources can be updated using a PATCH call on the resources endpoint. Only the fields to be changed should be included in the payload.

**Note:** When updating a provisioned resource, the API will automatically create an asynchronous `Update` action to propagate the change to actual services. The response will include a reference to the location of this action.

```
PATCH /api/v1/resources/{resource-guid}.json
```

**Possible request payload:**

```
{
	"resource": [
		"email_local": "some.projector",
		"password": "NewPassw0rd!",
		"display_name": "Some Projector",
		"resource_type": "Equipment"
	]
}
```

**Response headers for inactive resources:**

```
204 No Content
```

**Response headers for active resources:**

```
202 Accepted
Location: /api/v1/resources/{resource-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

### Deleting a calendar resource

Unlike mailboxes, active calendar resources must be "deleted" from the platform using a service action, in the same way domains are deleted, by posting an asynchronous `Delete` action for the resource.

**Deletion of an active calendar resource from the platform**

```
POST /api/v1/resources/{resource-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Delete"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/resources/{resource-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

Once the action completes, the inactive resource will automatically be deleted from the API records.

**Deleting an inactive calendar resource from the API records**

If a resource was never provisioned, the inactive record can be deleted from the API directly.

```
DELETE /api/v1/resources/{resources-guid}.json
```

**Response headers:**

```
204 No Content
```

## Usage reporting

In order to accurately bill an end customer for usage on a given domain, or provide them with projected billing numbers based on their current usage, an integrator may request a usage report via the API. This is a two step process. First, an asynchronous `Usage` action is created on the domain. Once that action completes, the results may be retrieved from the normal GET endpoint on the domain.

First, the usage action is created.

```
POST /api/v1/domains/{domain-guid}/actions.json
```

**Request payload:**

```
{
	"action": {
		"action": "Usage"
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/domains/{domains-guid}/actions/{action-id}
```

(See [Polling an asynchronous action](#polling-an-asynchronous-action))

Once the action is completed, the domain GET endpoint can be interrogated for the usage information. The endpoint will report usage for all editions which are currently in use, even on domains with multiple packages and multiple editions:

```
GET /api/v1/domains/{guid}.json
```

**Sample response for a Cloud Mail Standard domain**

```
{
	"domain": {
		"domain_name": "example.com",
		"domain_usage_report": {
			"state": "FINISHED",
			"data_as_at": "2018-02-19 13:00:00+0200"
			"edition_usage_reports": [
				{
			    	"edition_code": "CLM-STD-50-ARCH",
			    	"count": 21
			   },
			   {
			    	"edition_code": "CLM-STD-100-ARCH",
			    	"count": 8
			   }
			]
		}
	}
}
```

**Sample response for a Securemail Standard domain:**

```
{
	"domain": {
		"domain_name": "example.com",
		"domain_usage_report": {
			"state": "FINISHED",
			"data_as_at": "2018-02-19 13:00:00+0200"
			"edition_usage_reports": [
		       {
		        	"edition_code": "SM-STD",
		        	"count": 25
		       }
	      ]
      }
	}
}
```

**Sample response for a Mail Management Premium domain:**

```
{
	"domain": {
		"domain_name": "example.com",
		"domain_usage_report": {
			"state": "FINISHED",
			"data_as_at": "2018-02-19 13:00:00+0200"
			"edition_usage_reports": [
				{
					"edition_code": "MM-PREM",
					"count": 18
				}
			]
		}
	}
}
```

**Sample response for a domain where the latest usage report is still being compiled:**

```
{
	"domain": {
		"domain_name": "example.com",
		"domain_usage_report": {
			"state": "PENDING"
		}
	}
}
```

### Detailed usage reports

Starting in version 6.3, released on 2018-09-07, the API supports detailed usage reporting, which shows the usage counts for each edition, and includes an itemised list of mailboxes which comprise the count for that edition.

The detailed usage report is always generated by the normal usage reporting action, but is not displayed on the standard domain endpoint.

To retrieve a detailed usage report, first request a usage reporting action on the domain using the standard process. Please see the documentation section on [usage reporting](#usage-reporting) above for a thorough explanation of the process.

Once the usage reporting action has completed. The detailed report can be retrieved from the specialised domain usage endpoint.

**Sample request:**

```
GET /api/v1/domains/{domain-guid}/usage.json
```

**Sample response for a domain with SecureArchive:**

```
{
  "domain_usage_report": {
    "state": "finished",
    "data_as_at": "2018-09-07T09:36:09+0200",
    "edition_usage_reports": [
      {
        "edition_code": "AR-SEC",
        "count": 3,
        "billable_mailboxes": [
          "info@some-domain.com",
          "accounts@some-domain.com",
          "someone@some-domain.com"
        ],
        "warning": null,
        "problem": null,
        "class_of_service": null
      }
    ]
  }
}
```

**Sample response for a domain with Mail Management Standard:**

```
{
  "domain_usage_report": {
    "state": "finished",
    "data_as_at": "2018-09-07T09:35:27+0200",
    "edition_usage_reports": [
      {
        "edition_code": "MM-STD",
        "count": 2,
        "billable_mailboxes": [
          "user@domain.com",
          "another@domain.com"
        ],
        "warning": null,
        "problem": null,
        "class_of_service": null
      }
    ]
  }
}
```

**Sample response for a domain with Securemail Standard:**

```
{
  "domain_usage_report": {
    "state": "finished",
    "data_as_at": "2018-09-07T09:35:27+0200",
    "edition_usage_reports": [
      {
        "edition_code": "SM-STD",
        "count": 2,
        "billable_mailboxes": [
          "user@domain.com",
          "another@domain.com"
        ],
        "warning": null,
        "problem": null,
        "class_of_service": null
      }
    ]
  }
}
```

### Mailbox metadata

Since API release 2021-11-23.1, for some domains, the detailed usage reporting endpoint will return additional metadata for each billable mailbox. This data is collated in a separate metadata array in the root of the response, and will have an entry for each mailbox for which metadata is available. There is no guarantee that all mailboxes will have metadata, and it is also possible for metadata fields to vary between mailboxes, so no assumptions should be made as to the presence of particular fields. They should be treated as optional, and only processed when available.

**Sample response for a domain with Cloud Mail Plus and selected mailbox metadata:**

```
{
  "domain_usage_report": {
    "state": "finished",
    "data_as_at": "2022-01-11T09:35:00+0200",
    "edition_usage_reports": [
      {
        "edition_code": "CLM-PLUS-50-ARCH",
        "count": 2,
        "billable_mailboxes": [
          "user@domain.com",
          "another@domain.com",
          "third@domain.com"
        ],
        "warning": null,
        "problem": null,
        "class_of_service": null
      }
    ],
    "mailbox_metadata": [
      {
        "address": "user@domain.com",
        "fields": [
          {
            "field": "description",
            "value": "Some description"
          },
          {
            "field": "password_last_modified",
            "value": "20220107072235Z"
          }
        ]
      },
      {
        "address": "another@domain.com",
        "fields": [
          {
            "field": "password_last_modified",
            "value": "20210610110855Z"
          }
        ]
      }
    ]
  }
}
```

# Full workflow examples for advanced operations

## Polling an asynchronous action

Posting actions on domains or packages in the API always result in the creation of an asynchronous action. Some other actions on the API also implicitly create an asynchronous action.

Asynchronous actions can always be identified by the response headers they generate. Any interaction on the API which results in an asynchronous action, will always return with the `202 Accepted` HTTP header.

Once an asynchronous action is created, it is the responsibility of the API client to continue to poll the status of that action to determine when it is completed, or to report errors to its end user if the action has failed.

Any interaction with the API which results in an asynchronous action will return headers like this:

*For actions on domains*

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

*For actions on packages*

```
202 Accepted
Location: /api/v1/packages/{package-guid}/actions/{action-id}
```

The location returned is the unique URI of the action. This URI can then be polled by sending a `GET` request to that location.

```
GET /api/v1/domains/{domain-guid}
```

The API will respond with the present state of the action:

*A pending action*

```
{
    "state":"pending"
}
```

*A finished action*

```
{
    "state":"finished"
}
```

*A failed action*

```
{
    "state":"error",
    "errors": [
        "Array of error message strings",
        "Potentially multiple messages"
    ]
}
```

The client should continue to poll the state of any action until it is either in the `finished` or `error` state.

**Important notes:**

* Some legacy action in the API return a lowercase `location` header, rather than the standard `Location`. These can not be changed at this point, as it would break any existing integrations.

  Therefor, for client libraries which treat HTTP headers in a case sensitive way, the client is responsible for checking for the the uppercase or lowercase version of the `Location` header whenever a `202 Accepted` is received.

* Should an action fail (result in the `error` state), the client is required to report this to the user. Once the underlying issue causing the error has been created, the client must send another one of the same actions, either automatically, or after being prompted by the user. A failed action will remain in that state, and the API will never automatically retry it.

## Migrating a domain from one product to another

To migrate a domain from one product to another, a package representing the new product must first be created under the same company, with the `expect_migration` flag raised.

Next, the existing domain should be linked to the new package via the LINK package endpoint (see [Linking an existing domain to a package](#linking-an-existing-domain-to-a-package))

The API will evaluate the request to see if the domain is eligible to migrate to the new product. There are some cases in which migration may be rejected:

* The domain currently has active mailboxes, and the new product does not include the service that mailboxes are provisioned on. In this case the mailboxes must be deleted first

* There are active mailboxes, and the new product lacks classes of service which can accommodate all the current mailboxes. In this case, mailboxes need to be downgraded to classes of service which have smaller or equal storage to a class in the new product, as well as a smaller or equal additional feature set. 

  **Example**: Migrating from Cloud Mail Legacy or Cloud Mail Standard to Cloud Mail Lite will not be allowed if there are mailboxes larger than 2GB.

Ideally, integrators should design with these limitations in mind, and warn the end user before they attempt to make such changes, rather than relying on errors from the API, though the API will provide appropriate error feedback in these instances.

If the migration is accepted, the API will automatically unlink the previous package from the domain.

**Important**: When the previous package is unlinked, any services which are not included in the new package will be decommissioned on the backing services **immediately**. Integrators are encouraged to warn users of this, to prevent unexpected data loss stemming from users not understanding that step.

In cases where this happens, the API will respond to the LINK domain call with a 202 Accepted, which links to the Prune action for the services which are no longer needed. Otherwise, the API will respond with a 204 No Content.

The domain will then be in the `inactive` state, and provisioning it proceeds as usual for a domain linked to a package. (See [Configuring the service fields on a domain](#configuring-the-service-fields-on-a-domain) and [Provisioning a domain](#provisioning-a-domain)) 

### Migrating from Cloud Mail Legacy to Cloud Mail Standard with Branding

In this example, an existing client on Cloud Mail Legacy want to migrate to the new Cloud Mail Standard product, and also add Branding

**Step 1: Create Cloud Mail Standard with Branding package**

Make sure to raise the `expect_migration` flag.

*Request*

```
POST /api/v1/ous/{company-guid}/packages
```

*Payload*

```
{
    "package": {
        "code":"CLM-STD-B",
        "expect_migration": true
    }
}
```
*Response headers*

```
201 Created
location: /api/v1/packages/{cloud-mail-package-guid}
```

**Step 2: Link the domain to the new package**

*Request*

```
LINK /api/v1/packages/{cloud-mail-package-guid}/domains/{cloud-mail-domain-guid}
```

*Response headers*

```
204 No Content
```

**Note**

At this point, the existing domain will have switched from the `active` to the `inactive` state. This is expected, as the domain needs to be configured and "reprovisioned" on the new services. The domain will not have been removed or changed on the backend, but the API will be updating any already-active mailboxes to appropriate new classes of service in the background.

**Step 3: Configure service fields for the domain**

Note the Securemail and Branding related fields which are now mandatory.

*Request*

```
PATCH /api/v1/domains/{cloud-mail-domain-guid}/servicefields.json
```

*Payload*

```
{
	"fields": {
   		"auth_method": "smtp",
   		"username": "smtp-username",
   		"password": "smtpPassw0rd!",
   		"admin_username": "some@person.com",
   		"admin_password": "adminPassw0rd!",
   		"admin_first_name": "Some",
   		"admin_last_name": "Person",
   		"service_provider": "other"
   	}
}
```

*Response headers*

```
204 No Content
```

**Step 4: Provision the new package**

*Request*

```
POST /api/v1/packages/{cloud-mail-package-guid}/actions
```

*Payload*

```
{
    "action": {
        "action": "Provision"
    }
}
```

*Response headers*

```
202 Accepted
Location: /api/v1/packages/{cloud-mail-package-guid}/actions/{action-id}
```

The action should now be polled using the standard process for polling asynchronous actions. See the documentation section on [polling an asynchronous action](#polling-an-asynchronous-action) for full details.

### Migrating from Cloud Mail Plus with Branding to Securemail Standard

In this example, a client on Cloud Mail Plus with Branding decides to host their mail elsewhere, but retain the Securemail Standard service for email security.

**Step 1: Create Securemail Standard package**

Make sure to raise the `expect_migration` flag.

*Request*

```
POST /api/v1/ous/{company-guid}/packages
```

*Payload*

```
{
    "package": {
        "code":"SM-STD",
        "expect_migration": true
    }
}
```
*Response headers*

```
201 Created
location: /api/v1/packages/{securemail-package-guid}
```

**Step 2: Link the domain to the new package**

*Request*

```
LINK /api/v1/packages/{securemail-package-guid}/domains/{domain-guid}
```

*Response headers*

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

The returned action is the automated prune action to remote the Zimbra service, which is no longer needed. The action should now be polled using the standard process for polling asynchronous actions. See the documentation section on [polling an asynchronous action](#polling-an-asynchronous-action) for full details.

Once the action is complete, the Zimbra service will be decommissioned, and the new package is ready to be configured.

**Note**

At this point, the existing domain will have switched from the `active` to the `inactive` state. This is expected, as the domain needs to be configured and "reprovisioned" on the new services. In this case, the domain has been removed from Zimbra, but no changes have been made to the Securemail or Branding services, as they are both included in the new package.

The domain should now be configured and provisioned as normal. which will reconfigure the domain on Securemail and Branding with settings appropriate for the new package.

**Step 3: Configure service fields for the domain**

Note the full Securemail and Branding related fields which are now mandatory.

*Request*

```
PATCH /api/v1/domains/{domain-guid}/servicefields.json
```

*Payload*

```
{
	"fields": {
	    "mail_delivery_location": "some.host.com",
   		"auth_method": "custom",
   		"admin_username": "some@person.com",
   		"admin_password": "adminPassw0rd!",
   		"admin_first_name": "Some",
   		"admin_last_name": "Person",
   		"service_provider": "office365"
   	}
}
```

*Response headers*

```
204 No Content
```

**Step 4: Provision the new package**

*Request*

```
POST /api/v1/packages/{securemail-package-guid}/actions
```

*Payload*

```
{
    "action": {
        "action": "Provision"
    }
}
```

*Response headers*

```
202 Accepted
Location: /api/v1/packages/{cloud-mail-package-guid}/actions/{action-id}
```

The action should now be polled using the standard process for polling asynchronous actions. See the documentation section on [polling an asynchronous action](#polling-an-asynchronous-action) for full details.

## Checking if a domain can be migrated to a new product and how the migration will affect mailbox allocations

The API accepts the OPTIONS method on the same endpoint used to link packages and domains with the LINK method. In this case, the API will perform a dry run of the link process. If the operation can not be completed, the same failure message that would have resulted from the LINK method will be returned. If the operation can proceed, the `204 No Content` response will be returned.

If the operation can be completed, and the migration is between two products with mailbox editions, a `200 OK` response will be returned, along with a JSON report showing the mappings between editions and the number of mailboxes that will be mapped from each edition. This allows integrators to provide end users with a preview of how mailbox editions will be affected by the migration.

### Checking if a domain with Securemail Standard can be migrated to Securemail Premium

```
OPTIONS /api/v1/packages/{premium-package-guid}/domains/{domain-guid}
```

*Response headears*
```
204 No content
```

As with all API endpoints where specific data can is not required, the `204 No content` response here indicates that the LINK method on the same endpoint would succeed, and the domain can by migrated to this package.

### Checking if a domain with Securemail Standard can be migrated to Cloud Mail Standard

```
OPTIONS /api/v1/packages/{cloud-mail-package-guid}/domains/{domain-guid}
```

*Response headers*
```
204 No content
```

### Checking if a domain with Cloud Mail Standard can be migrated to Securemail Standard with mailboxes still active

```
OPTIONS /api/v1/packages/{securemail-package-guid}/domains/{domain-guid}
```

*Response headers*
```
409 Conflict
```

*Response payload*

```
{
  "code": 409,
  "message": "This migration will result in the removal of a service on which active mailboxes depend. The mailboxes must be removed first.",
  "errors": null
}
```

In this case, the migration can not proceed right away, because there are active mailboxes on the domain, which need to be removed.

### Checking if a domain with Cloud Mail Standard can be migrated to Securemail Standard with no active mailboxes remaining

```
OPTIONS /api/v1/packages/{securemail-package-guid}/domains/{domain-guid}
```

*Response headers*
```
204 No content
```

### Checking if a domain with legacy Cloud Mail can be migrated to Cloud Mail Plus

```
OPTIONS /api/v1/packages/{plus-package-guid}/domains/{domain-guid}
```

*Response headers*
```
200 OK
```

*Response payload*

```
{
  "mailbox_edition_map": [
    {
      "current_edition": "SYN-CMS-BASIC-02",
      "new_edition": "CLM-LT",
      "count": 1
    },
    {
      "current_edition": "SYN-CM-PREM-25-ARCH",
      "new_edition": "CLM-PLUS-50-ARCH",
      "count": 2
    },
    {
      "current_edition": "SYN-CM-PREM-100-ARCH",
      "new_edition": "CLM-PLUS-100-ARCH",
      "count": 1
    },
    {
      "current_edition": "SYN-CMS-PREM-50",
      "new_edition": "CLM-PLUS-50-ARCH",
      "count": 1
    }
  ]
}
```

In this case, both products have mailbox editions, so the API returns a report showing the upgrade path for each edition in the old package to an edition in the new package.

**Note**: In some cases, such as migrating from a legacy package to a new package, several old editions may be rolled up into one new edition. In such cases, the API reports the counts separately for each **old** edition. It is the responsibility of the integrator to calculate the totals for each new edition, if that information is to be presented to the end user.

## Moving a domain from one company to another

To move a domain from one company to another, a new package representing the same product, or a new product, must first be created under the company to which the domain should move, with the  `expect_domain_move` flag raised. If you also wish to move the domain to a different product at the same time, the  `expect_migration` flag should also be raised. It is safe to always raise the `ecpect_migration` flag for domain moves, to simplify implementation.

Next, the existing domain should be linked to the new package via the LINK package endpoint (see [Linking an existing domain to a package](#linking-an-existing-domain-to-a-package))

The API will evaluate the request to see if the domain is eligible to move to the new company and product.

If the domain move is accepted, the API will automatically unlink the previous package in the old company from the domain, and the domain will move to the new company.

**Important**: If also moving to a different product, when the previous package is unlinked, any services which are not included in the new package will be decommissioned on the backing services **immediately**. Integrators are encouraged to warn users of this, to prevent unexpected data loss stemming from users not understanding that step.

The API will respond to the LINK domain call with a 202 Accepted, which links to a Move action on the domain. This action wil start executing immediately, and will complete all preparatory steps in the back end services which are required to move the domain.

The domain will then be in the `inactive` state, and provisioning it proceeds as usual for a domain linked to a package. (See [Configuring the service fields on a domain](#configuring-the-service-fields-on-a-domain) and [Provisioning a domain](#provisioning-a-domain)) 

### Example

In this example, a domain currently provisioned under Foo Industries, with the hypothetical API GUID `FOO-CORP` needs to move to Bar Industries, with the hypothetical API GUID `BAR-CORP`. In this case, the domain currently has Cloud Mail Plus, and the customer wishes to continue to use that product. If a different product is to be used, remember to also raise the `expect_migration` flag on the package when creating it.

**Step 1: Create the new Cloud Mail Plus package under Bar Industries**

Make sure to raise the `expect_domain_move` flag.

**NOTE**: If the client wants to move to a new product, you should also raise the `expect_migration` flag. It is safe to do so anyway, to simplify your implementation. 

*Request*

```
POST /api/v1/ous/{BAR-CORP}/packages
```

*Payload*

```
{
    "package": {
        "code":"CLM-PLUS",
        "expect_domain_move": true
    }
}
```

*Response headers*

```
201 Created
location: /api/v1/packages/{NEW-CLOUD-MAIL-PLUS-GUID}
```

**Step 2: Link the domain to the new package**

*Request*

```
LINK /api/v1/packages/{NEW-CLOUD-MAIL-PLUS-GUID}/domains/{domain-guid}
```

*Response headers*

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

The returned action is the automated Move action to prepare the back end services for the domain to be moved. The action should now be polled using the standard process for polling asynchronous actions. See the documentation section on [polling an asynchronous action](#polling-an-asynchronous-action) for full details.

**Note**

At this point, the existing domain will have switched from the `active` to the `inactive` state. This is expected, as the domain needs to be configured and "reprovisioned" on the new company, potentially with different service fields.

The domain should now be configured and provisioned as normal. which will update any back end service information to the configuration the new owners of the domain have selected.

**Step 3: Configure service fields for the domain**

*Request*

```
PATCH /api/v1/domains/{domain-guid}/servicefields.json
```

*Payload*

```
{
	"fields": {
   		"auth_method": "smtp",
   		"admin_username": "some@person.com",
   		"admin_password": "adminPassw0rd!",
   		"admin_first_name": "Some",
   		"admin_last_name": "Person",
   		"service_provider": "other"
   	}
}
```

*Response headers*

```
204 No Content
```

**Step 4: Provision the new package**

*Request*

```
POST /api/v1/packages/{NEW-CLOUD-MAIL-PLUS-GUID}/actions
```

*Payload*

```
{
    "action": {
        "action": "Provision"
    }
}
```

*Response headers*

```
202 Accepted
Location: /api/v1/packages/{NEW-CLOUD-MAIL-PLUS-GUID}/actions/{action-id}
```

The action should now be polled using the standard process for polling asynchronous actions. See the documentation section on [polling an asynchronous action](#polling-an-asynchronous-action) for full details.

# Changes between the legacy SYNAQ API and the current structure

This section contains compatibility notes for integrators who wish to upgrade to the new API structure from the legacy structure.

## New product bundles

The existing SYNAQ product suite (product codes starting with SYN) has been replaced by a new bundle structure. For details on the new bundles, please see the section on [Product bundles](#product-bundles)

It is recommended that integrators cease the sale of old products as soon as possible, and sell the new bundles to new clients. Please contact your SYNAQ account manager for sales collateral and further details on the new bundles.

It is also recommended that existing customers be upgraded to the new bundles as soon as possible, or at the end of their contract terms. See the section on [Migrating a domain from one product to another](#migrating-a-domain-from-one-product-to-another) for technical details of how this can be achieved.

## Product group codes and edition codes

It is no longer necessary to provide the `product_group_code`or `editons` fields when creating a package which is an instance of the new product bundles. These fields are deprecated, but still available for legacy products.

## Single domain products

All of the new product bundles are implemented as packages which can only be linked to a single domain. In some limited use cases, integrators are still permitted to link multiple packages to a domain, but never more than one domain to any given package.

This was done to simplify the process to cancel only some domains in a subscription. To implement a client with multiple domains, create a separate package for each domain.

These separate packages and domains can then be managed independently of each other, negating the need for the legacy partial cancellation workflow.

## Partial cancellations

The partial cancellation workflow which existed in the legacy API is still supported, and will work for legacy products with multiple domains or packages which are linked together.

This process is deprecated, and omitted from this version of the documentation to avoid confusion.

When clients which to partially cancel existing services, we recommend the change be implemented as a migration to a new product which does not include the unwanted service instead.

If you need to perform the old workflow for maintenance reasons, please consult the [Legacy API documentation](#legacy-api-documentation)

## Unlinking domains from packages

The API still allows unlinking domains from packages, but as the partial cancellations workflow is now deprecated, that use case is omitted from this document.

The recommended way to remove unwanted services from a domain in the new structure is to migrate the domain to a product bundle which does not include the unwanted service. Please see [Migrating a domain from one product to another](#migrating-a-domain-from-one-product-to-another) for details.

If you need to perform the old workflow for maintenance reasons, please consult the [Legacy API documentation](#legacy-api-documentation)

## Immediate provisioning of domains

The immediate provisioning workflow, triggered by raising the `provision_immediately` flag on a domain. which was implemented for legacy Cloud Mail domains, is still available for the legacy product, but is deprecated in this version of the API, and excluded from this version of the documentation to avoid confusion. This is because all products (including the new Cloud Mail bundles) now require service field configuration before provisioning.

If you need to perform the old workflow for maintenance reasons, please consult the [Legacy API documentation](#legacy-api-documentation)

The immediate provisioning workflow for **mailboxes** is still supported and available.
