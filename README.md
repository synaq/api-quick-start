# SYNAQ API Quick Start Guide

Updated 2018-03-27

## Introduction

The SYNAQ API allows resellers integrated with it to directly manipulate customer data, and provision and manage SYNAQ services under domains for their customers.

Before getting into concrete usage examples for our various products, we need to define a few basic concepts which apply throughout all API workflows.

## Basic concepts

### Products

Products are combinations of physical services sold by SYNAQ. Examples of products include CouldMail, our cloud based email hosting service, Securemail, our email security service, etc. Products are purchased via the API by creating packages, as described below. A package is an instance of a given product.

### Editions

Most of SYNAQ's products offere different classes or levels of service, to allow the package to be tailored to the end customer's specific requirements and budget. When a package is created, it must be assigned one or more editions, depending on the product. Plan based products (Securemail, Branding, Archive) require one edition. Mailbox based products such as CloudMail and Continuity may have any number of editions assigned to their packages, and individual mailboxes are then assigned to specific editions to determine their class of service.

### Organisational Units (OUs)

An OU is the most basic structure which can be created by the API. OUs represent a reseller's customers, either as "sub-resellers", who can in turn sell to their own clients, or "companies", which are direct clients of the reseller.

The API allows creation of organisational units, under which packages and domains must then be created and linked to each other in order to provision services.

### Packages

A package is an instance of a given SYNAQ product under a particular organisational unit. For example, if a client wishes to buy the SYNAQ Securemail product, an OU would be created to represent that client, and then a SYNAQ Securemail package would be created under that OU.

When creating packages, one or more editions may be specified, to determine the actual class or level of service offered under the package.

### Domains

A domain is a representation in the API of an actual customer domain. Domains exist underneath OUs, and must be linked to one or more packages in order to be provisioned. The packages linked to a given domain will determine which actual services are provisioned for that domain on the SYNAQ platform.

### Mailboxes

For products where mailboxes can be managed directly via the API (SYNAQ CloudMail and SYNAQ Continuity at the time of this writing), mailboxes in the API represent actual mailboxes on those services. The API allows creation of mailboxes underneath domains, and then allows the mailboxes to be provisioned onto the actual backing services on the SYNAQ platform.

### Service fields

Some SYNAQ products require additional configuration for provisioning to be possible. This includes information like the delivery destination to use for inbound Securemail domains, the authentication mechanism for outbound Securemail domains, the authentication mechanism to use for Branding domains, etc.

Service fields are managed via the API through a PATCH call made on a domain object. This must be done after all desired packages are linked to the domain, and before attempting to provision the domain though a provisioning action.

### Asynchronous actions

The SYNAQ API abstracts a powerful services bus mechanism used on the SYNAQ platform to queue and complete actual provisioning tasks on the services on our platform. As some tasks may require a short time to complete, or wait in the queue at particularly busy times, all of these services bus tasks are represented in the API by asynchronous actions.

Asynchronous actions will generally complete very quickly, but can be delayed in especially busy situations, or for services where some manual intervention might be required on the SYNAQ platform. In general, all asynchronous actions on products other than SYNAQ Archive should complete within a minute or less, and almost always complete within a few seconds.

When asynchronous actions are created, the API returns their location to the client in an HTTP header. This location can be used to poll the action and determine its state. When a problem occurs, the action will also report the reason for the failure, which can be relayed to SYNAQ support for assistance.

Full documentation on the use of asynchronous actions may be found in the usage example section of this document.

### Domain (and mailbox) actions

The most important asynchronous actions in the API are actions on domains and mailboxes. These can be triggered explicitly by creating them on the domain or mailbox actions endpoint in the API, and are sometimes triggered implicitly by certain endpoints, or by specific configuration settings on some endpoints. In cases where this may happen, this document will highlight that possibility.

The most important and frequently used domain and mailbox action is the `Provision` action. This initiates the steps needed to actually provision an already configured domain (or mailbox) on SYNAQ's platform.

## Prerequisites

To use the API, an integrator requires the top level GUID assigned to their reseller, and a valid API key. These will be provided by SYNAQ upon request. For new resellers, we will generally only allocate credentials for our staging environment, enabling the integrator to develop and test their integration client without the possibility of affecting production services or incurring billing.

Once the integrator is satisfied that their integration is stable and ready to serve their customers, we will provide access credentials for the production API.

## Online API documentation and development sandbox

SYNAQ supplies an online API documentation and sandbox tool. The tool lists all possible calls to the SYNAQ API, together with their expected payloads, and possible responses.

By clicking on the `Sandbox` tab under each documented endpoint, the user may also make a live request to the API for that call directly from the tool. This was intended to make the process of gaining sample payloads and responses for the API easier, and also to aid in troubleshooting.

The tool is available on our staging API infrastructure at this URL:

<https://sta-q.synaq.com/api-doc>

Please log in using your user interface credentials, not your API key. This will be provided along with other staging configuration information.

## Developer Support

The SYNAQ development team is available to support developers with integration partners via email. Please contact the team directly on <dev@synaq.com> for assistance.

## Usage examples

### Request Authentication

All requests to the SYNAQ API must be authenticated using SWSS authentication, injected into the HTTP headers. These headers are required in all requests:

```
synaq-api-user: {reseller API user name}
synaq-api-salt: {current UNIX timestamp}
synaq-api-hash: {user name, API key and salt, concatenated and hashed in SHA1}
```

The user name and API key are provided by SYNAQ. The salt is a standard UNIX timestamp**, and the hash is calculated by concatenating the username, API key and salt, and hashing them using the SHA1 hashing algorithm.

** While a standard timestamp will suffice, it is important that timestamps not be reused, to prevent replay attacks. If you expect your API client to make more than one request per second, we advise using a modified timestamp which includes milliseconds or microseconds to increase granularity.

*PHP Hash Generation Example*

```
$username = 'client@client.com';
$apiKey = 'some-long-api-key-string-provided-by-us';
$salt = microtime(true) * 10000;
$hash = sha1($username . $apiKey . $salt);
```

### OU Creation: Creating a company underneath an existing reseller

Create a company titled "Acme Ltd", belonging to the authenticated reseller.

```
POST /api/v1/ous/{reseller-guid}/companies.json
```

*reseller-guid:* The root level reseller GUID provided by SYNAQ, or a sub-reseller GUID, which can be obtained by first creating a sub-reseller** via the API.

*Request payload:*

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

The `title` field on the OU, and `line_1`, `city`, `postal_code` and `country` on the `physical_address` object are all requered fields. All other fields are optional.

*Response headers:*

```
201 Created
location: /api/v1/ous/{ou-guid}
```

Visiting the location will show the basic OU information. The OU GUID can also be extracted from the `location` header.

** To create a sub-reseller, follow the same process as creating as for company creation, but target the appropriate endpoint:

```
POST /api/v1/ous/{reseller-guid}/subresellers.json
```

### Package Creation: Look up all possible package combinations under a given company

```
OPTIONS /api/v1/ous/{company-guid}/packages.json
```

The response will show all possible product and edition combinations for new packages under the given company. Due to this response being quite detailed and lengthy, it has been ommitted from this document. Please use the API sandbox tool to generate a sample response for your own test company.

This response can be used to dynamically generate forms with available products on your client system.

### Package Creation: Create a new CloudMail package

Create an instance of the CloudMail product with 2 mailbox types; Basic 2GB and Premium 25GB

```
POST /api/v1/ous/{company-guid}/packages.json
```

*Request payload:*

```
{
  "package": {
    "code": "SYN-CMS",
    "editions": [
      {
        "code": "SYN-CMS-BASIC-02"
      },
      {
        "code": "SYN-CM-PREM-25-ARCH"
      }
    ]
  }
}
```

*Response headers:*

```
201 Created
location: /api/v1/packages/{package-guid}
```

*Note:* An empty array can be specified for the editions parameter above on the CloudMail product only. In such cases, all default allowed editions will be enabled for the package.

For all other products, an edition must be selected at package creation time.

### Package Creation: Create a new Securemail Bidirectional package

Create a Securemail bidirectional package, note the edition code "SYN-PIN-SEC-BOTH". Securemail is a plan based product, so an edition code must always be specified.

```
POST /api/v1/ous/{company-guid}/packages.json
```

*Request payload:*

```
{
    "package":
    {
        "code":"SYN-PIN-SEC",
        "editions":
        [
            {"code":"SYN-PIN-SEC-BOTH"}
        ]
    }
}
```

*Response headers:*

```
201 Created
location: /api/v1/packages/{package-guid}
```

### Package Creation: Create a new Branding package

Create a Branding package, note the edition code "SYN-PIN-BRD", Branding is a plan based product, so must always have an edition code specified, similar to Securemail.

```
POST /api/v1/ous/{company-guid}/packages.json
```

*Request payload:*

```
{
    "package":
    {
        "code":"SYN-PIN-BRD",
        "editions":
        [
            {"code":"SYN-PIN-BRD"}
        ]
    }
}
```

*Response headers:*

```
201 Created
location: /api/v1/packages/{package-guid}
```

### Domain Creation: Create a new standalone domain which can be linked to an existing package

```
POST /api/v1/ous/{company-guid}/domains.json
```

*Request payload:*

```
{
  "domain": {
    "domain_name": "acme.com"
  }
}
```

*Response headers:*

```
201 Created
location: /api/v1/domains/{domain-guid}
```

### Domain Creation: Linking an existing domain to a package

Before packages or domains can be provisioned, or have their service fields configured for products where that is required, they must be linked in the API.

```
LINK /api/v1/packages/{package-guid}/domains/{domain-guid}.json
```

*Response headers:*

```
204 No Content
```

### Domain Creation: Creating a new domain and linking it to an existing package automatically

To simplify the process of creating domains and linking them in implementations where a package is likely to only require a single domain, the API provides a convenience method where a domain is created underneath a package.

In reality, the domain is still owned by the OU which owns the package, but using this method, the domain is linked to the package immediately, negating the need for the intermediary linking step.

```
POST /api/v1/packages/{package-guid}/domains.json
```

*Request payload:*

```
{
  "domain": {
    "domain_name": "acme.com"
  }
}
```

*Response headers:*

```
201 Created
location: /api/v1/domains/{domain-guid}
```

### Domain Creation: Configuring the service fields on a domain

Most products require additional service information to be configured before a domain can be provisioned. Which information is required is determined by the package(s) linked to the domain, so linking must be completed before these steps.

To determine which fields require information, the general use GET endpoint for domains may be used with the given domain GUID, and the `fields` object may be inspected.

**Note**: The endpoint also exposes a legacy `service_fields` object, which was used in a previous mechanism for retreiving information passed back from the services. Please do no confuse that object for `fields` as described above.

```
GET /api/v1/domains/{domain-guid}.json
```

**Sample Response for a bidirectional Securemail domain (abbreviated):**

```
{
  "guid": "domain-guid",
  "domain_name": "some-domain.com",
  "fields": {
    "mail_delivery_location": null,
    "mail_delivery_alternative": null,
    "auth_method": "smtp",
    "username": null,
    "password": null
  }
}
```

This shows required fields for the selected package combination, and their current values, if any. Once the fields are known, or using a predefined set of fields for given products in the client, the service information should be requested from the end user, and then updated on the API using the special PATCH endpoint for service fields on the given domain


```
PATCH /api/v1/domains/{domain-guid}/servicefields.json
```

**Reqeust payload:**

```
{
	"fields": {
   		"mail_delivery_location": "smtp.some-domain.com",
   		"mail_delivery_alternative": "smtp2.some-domain.com",
   		"auth_method": "smtp",
   		"username": "smtp-username",
   		"password": "smtpPassw0rd!"
   	}
}
```

**Response headers:**

```
204 No Content
```

### Domain Creation: Provisioning a domain

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

### Domain Creation: Provisioning a package

In cases when multiple domains are linked to a package, or where a new package is being added to an existing domain, the API also permits a provision action to be created on a package rather than a domain. The process for doing this is the same as for provisioning actions created directly on domains, but using the package actions endpoints.

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

The polling process for actions on packages is similar to that shown for domains above.

### Domain Creation: Creating a new domain, linking it to an existing CloudMail package, and provisioning it automatically

For the CloudMail product, no service field configuration is required, so to streamline the provisioning process even further, especially for integrations where only CloudMail is required, the API provides a special `provision_immediately` flag on the endpoint which allows domains to be created and linked to a package.

If the flag is raised, the domain will be created and linked to the package as per the simplified workflow, but a provision action will also be immediately created.

In this case, the API will not return the usual 201 Created with a location header referring to the domain itself, but a 202 Accepted, with a location header referring to the automatically created domain provisioning action.

The GUID for the actual domain may be inferred by parsing the location of the provisioning action.

```
POST /api/v1/packages/{package-guid}/domains.json
```

*Request payload:*

```
{
  "domain": {
    "domain_name": "acme-cloudmail.com",
    "provision_immediately": true
  }
}
```

*Response headers:*

```
202 Accepted
location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

### Domain maintenance: Closing a domain

In some cases, an integrator may wish to temporarily suspend the operation of a domain. This is usually needed by integrators who want to enforce credit control on end customers by temporarily denying service, without completely tearing down the provisioned services.

A closed domain will not process incoming mail, nor allow outgoing mail to be relayed.

To close a domain, create an asynchronous `Close` action on the domain actions endpoint. Poll the action as usual.

```
POST /api/v1/packages/{package-guid}/actions.json
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
location: /api/v1/packages/{package-guid}/actions/{action-id}
```

### Domain maintenance: Reactivating a domain

A closed domain may be reactivated (returned to service) by using the asynchronous `Activate` action.

```
POST /api/v1/packages/{package-guid}/actions.json
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
location: /api/v1/packages/{package-guid}/actions/{action-id}
```

### Domain maintenance: Deleting a domain

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

Once the action reports a state of `finished`, the domain will be in the `deleted` state, and thus eligible for final deletion from the API records using the `DELETE` endpoint.

```
DELETE /api/v1/domains/{domain-guid}.json
```

**Response headers:**

```
204 No Content
```

**Notes**:

* The `DELETE` action is only permitted for domains in the `inactive` or `deleted` states. A domain in any other state can not be deleted from the API's records.

* For products which expose mailboxes directly as API objects, such as CloudMail and Continuity, a domain may not be deleted if it still containeds mailboxes in provisioned states. For this reason, the API will reject an asynchronous `Delete` action on any domain where provisioned mailboxes are still present. To delete such a domain, the mailboxes must be deleted first. Please refer to the mailbox management section below.

### Package maintenance: Deleting a package

If all of the domains on a package are either in the `deleted` state, or if the package has no more domain records linked to it, the package itself may be deleted by a `DELETE` endpoint.

```
DELETE /api/v1/packages/{package-guid}.json
```

**Response headers:**

```
204 No Content
```

### Organisation Maintenance: Deleting an organisation

If a company no longer has any active packages associated with it, its records may also be deleted form the API using a delete endpoint.

```
DELETE /api/v1/ous/{ou-guid}.json
```

**Response headers:**

```
204 No Content
```

### Mailbox Creation: Creating a mailbox under a domain

For products where mailboxes are directly exposed via the API, such as CloudMail and Continuity, a mailbox may be created and provisioned under a domain once the domain has been provisioned.

**Notes:**

* Only the `email_local`, `password` and `last_name` fields are required. Many other fields are available. Please see the API documentation for a full description of all available fields.
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
		"package_edition_code": "SYN-CM-PREM-25-ARCH"
	}
}
```

**Response headers:**

```
201 Created
Location: /api/v1/mailboxes/{mailbox-guid}
```

### Mailbox Creation: Provisioning a mailbox

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

### Mailbox Creation: Creating a mailbox under a domain and provisioning it immediately.

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
		"package_edition_code": "SYN-CM-PREM-25-ARCH",
		"provision_immediately": true
	}
}
```

**Response headers:**

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

### Mailbox Maintenance: Updating a mailbox

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

### Mailbox Maintenance: Suspending a mailbox

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

### Mailbox Maintenance: Closing a mailbox

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

### Mailbox Maintenance: Reactivating a mailbox

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

### Mailbox Maintenance: Unlocking a mailbox

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

### Mailbox Maintenance: Changing the edition (class of service) of a mailbox

From time to time, it may be necessary to change the package edition associated with a mailbox. This has the effect of also changing the actual class of service of a provisioned mailbox.

This may be used to upgrade a mailbox, especially in cases where an integrator wishes to up-sell users onto a more premium package, or to downgrade a user, in cases where end customers might want to manage their spend by rationalising the service classes offered to their internal users.

If the mailbox is already provisioned, this call will automatically create an asynchronous update action. The change to the actual service will be visible when that action is completed.

```
PATCH /api/v1/mailboxes/{mailbox-guid}/packages/{package-guid}/edition.json
```

**Request payload**:

```
{
	"edition": {
		"code": "SYN-CM-PREM-100-ARCH"
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

**Note:** Any valid edition code on the package may be specified. In the above example, a mailbox is upgraded to the premium 100GB option. It could also have been downgraded to a basic 2GB mailbox by using the appropriate edition code, for example `SYN-CMS-BAISC-02`. The valid editions on a package may be verified by using the GET call on the package itself.

### Mailbox Maintenance: Delete a mailbox

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

***Response headers for active mailboxes:***

```
202 Accepted
Location: /api/v1/mailboxes/{mailbox-guid}/actions/{action-id}
```

**Note:** Unlike other asynchronous actions on API objects. The delete action will never present a `finished` state. This is because the one-step mailbox removal process also deletes the actual mailbox record in the case of a successful delete action, deleting the action along with it. Thus, the action may be polled while it is in progress, but when finished, polling it will return a `404 Not Found`, as will any attempt to retrieve mailbox records.

So, for this special use case, the deletion may be assumed to be complete as soon as a poll to the action or the mailbox itself returns not found.

### Billing: Request a usage count for a domain

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

Once the action is completed, the domain GET endpoint can be interrogated for the usage information. The endpoint will report usage for all editions which are currently in use, even on domains with multiple packages and multiple editions:

```
GET /api/v1/domains/{guid}.json
```

**Sample response for a CloudMail domain**

```
{
	"domain": {
		"domain_name": "example.com",
		"domain_usage_report": {
			"state": "FINISHED",
			"data_as_at": "2018-02-19 13:00:00+0200"
			"edition_usage_reports": [
				{
			    	"edition_code": "SYN-CMS-BASIC-02",
			    	"count": 21
			   },
			   {
			    	"edition_code": "SYN-CM-PREM-25-ARCH",
			    	"count": 8
			   }
			]
		}
	}
}
```

**Sample response for a Securemail domain**

```
{
	"domain": {
		"domain_name": "example.com",
		"domain_usage_report": {
			"state": "FINISHED",
			"data_as_at": "2018-02-19 13:00:00+0200"
			"edition_usage_reports": [
		       {
		        	"edition_code": "SYN-PIN-SEC-BOTH",
		        	"count": 25
		       }
	      ]
      }
	}
}
```

**Sample response for a Mail Management Suite domain**

```
{
	"domain": {
		"domain_name": "example.com",
		"domain_usage_report": {
			"state": "FINISHED",
			"data_as_at": "2018-02-19 13:00:00+0200"
			"edition_usage_reports": [
				{
					"edition_code": "SYN-PIN-BDL-10",
					"count": 18
				}
			]
		}
	}
}
```

**Sample response for a domain where the latest usage report is still being compiled**

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

## Advanced use cases

The API supports advanced use cases not covered in this documentation. These are normally either for use by SYNAQ internally, or were developed for specific integrators. They are not covered here for the sake of brevity, but are available to all integrators on request. Please contact SYNAQ for more information on these use cases.