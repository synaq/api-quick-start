# Usage Reporting

Usage reports show the number of billable mailboxes per edition on a domain. They are used for accurate billing and can be used to generate projected billing numbers for end customers.

Usage reporting is a two-step process:

1. Trigger a `Usage` action on the domain
2. Retrieve the results from the domain GET endpoint (or the dedicated usage endpoint for detailed reports)

---

## Trigger a usage report

```
POST /api/v1/domains/{domain-guid}/actions.json
```

```json
{
  "action": {
    "action": "Usage"
  }
}
```

**Response:**

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

Poll the action until it reaches `finished`. See [Async Actions](async-actions.md).

---

## Retrieve usage summary

Once the action completes, the summary is available on the standard domain endpoint:

```
GET /api/v1/domains/{domain-guid}.json
```

**Sample response:**

```json
{
  "domain": {
    "domain_name": "acme.com",
    "domain_usage_report": {
      "state": "FINISHED",
      "data_as_at": "2024-03-15 09:00:00+0200",
      "edition_usage_reports": [
        {
          "edition_code": "{edition-code}",
          "count": 21
        },
        {
          "edition_code": "{edition-code}",
          "count": 8
        }
      ]
    }
  }
}
```

**While the report is still being compiled:**

```json
{
  "domain": {
    "domain_name": "acme.com",
    "domain_usage_report": {
      "state": "PENDING"
    }
  }
}
```

---

## Detailed usage report

The detailed report includes an itemised list of billable mailboxes per edition.

```
GET /api/v1/domains/{domain-guid}/usage.json
```

**Sample response:**

```json
{
  "domain_usage_report": {
    "state": "finished",
    "data_as_at": "2024-03-15T09:36:09+0200",
    "edition_usage_reports": [
      {
        "edition_code": "{edition-code}",
        "count": 3,
        "billable_mailboxes": [
          "info@acme.com",
          "accounts@acme.com",
          "someone@acme.com"
        ],
        "warning": null,
        "problem": null,
        "class_of_service": null
      }
    ]
  }
}
```

---

## Mailbox metadata

For some domains, the detailed usage endpoint also returns additional metadata per mailbox. This data is in a separate `mailbox_metadata` array and is optional — not all mailboxes will have metadata, and available fields vary.

**Sample response with metadata:**

```json
{
  "domain_usage_report": {
    "state": "finished",
    "data_as_at": "2024-03-15T09:35:00+0200",
    "edition_usage_reports": [
      {
        "edition_code": "{edition-code}",
        "count": 2,
        "billable_mailboxes": ["user@acme.com", "another@acme.com"]
      }
    ],
    "mailbox_metadata": [
      {
        "address": "user@acme.com",
        "fields": [
          { "field": "description", "value": "Finance team" },
          { "field": "password_last_modified", "value": "20240107072235Z" }
        ]
      }
    ]
  }
}
```

Do not assume any particular metadata field is present. Treat all metadata fields as optional.
