# Organisational Units (OUs)

OUs represent your customers in the API hierarchy. A company is a direct customer of your reseller — packages and domains are created under companies.

---

## List OUs

```
GET /api/v1/ous.json
```

Returns all OUs in your reseller scope. Supports rich filtering:

| Parameter      | Description |
|----------------|-------------|
| `page`         | Page number (default: 1) |
| `limit`        | Items per page (default: 5) |
| `title`        | Filter by OU name |
| `slug`         | Filter by URL-friendly slug |
| `search`       | Free-text search across title and other fields |
| `parent_guid`  | Return only OUs directly under this parent |
| `order_by`     | Field to sort by (e.g. `title`) |
| `sort`         | Sort direction: `asc` or `desc` |
| `exact_match`  | `1` to require exact matches on text filters |

**Example — list companies under your reseller:**

```
GET /api/v1/ous.json?parent_guid={reseller-guid}&limit=25
```

**Sample response:**

```json
{
  "page": 1,
  "limit": 25,
  "pages": 1,
  "total": 3,
  "_embedded": {
    "ous": [
      {
        "guid": "COMPANY-GUID-1",
        "title": "Acme Ltd",
        "type": "company"
      }
    ]
  }
}
```

---

## Get a single OU

```
GET /api/v1/ous/{ou-guid}.json
```

---

## Create a company

```
POST /api/v1/ous/{reseller-guid}/companies.json
```

**Request payload:**

```json
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

Required fields: `title`, `physical_address.line_1`, `physical_address.city`, `physical_address.postal_code`, `physical_address.country`. All others are optional.

**Response:**

```
201 Created
Location: /api/v1/ous/{ou-guid}
```

The new company's GUID can be extracted from the `Location` header.

---

## Update an OU

```
PATCH /api/v1/ous/{ou-guid}.json
```

Include only the fields you want to change:

```json
{
  "ou": {
    "title": "Acme Corporation",
    "phone_number": "0117654321"
  }
}
```

**Response:**

```
204 No Content
```

---

## Delete an OU

OUs can only be deleted if they have no active packages associated with them.

```
DELETE /api/v1/ous/{ou-guid}.json
```

**Response:**

```
204 No Content
```
