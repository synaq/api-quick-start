# Pagination

All list endpoints in the SYNAQ API return paginated responses using the HATEOAS envelope format.

---

## Response envelope

```json
{
  "page": 1,
  "limit": 10,
  "pages": 5,
  "total": 47,
  "_embedded": {
    "mailboxes": [
      { ... },
      { ... }
    ]
  }
}
```

| Field       | Description |
|-------------|-------------|
| `page`      | Current page number (1-based) |
| `limit`     | Items per page |
| `pages`     | Total number of pages |
| `total`     | Total number of matching items |
| `_embedded` | Object containing the result array, keyed by resource name (`mailboxes`, `dls`, `resources`, etc.) |

---

## Requesting a specific page

Append `page` and `limit` query parameters:

```
GET /api/v1/mailboxes.json?domain_guid={guid}&exact_match=1&page=2&limit=25
```

---

## Iterating all results

```python
def get_all(base_url, path, params, headers):
    results = []
    page = 1
    while True:
        params['page'] = page
        response = requests.get(base_url + path, params=params, headers=headers).json()
        # Extract the embedded resource (key varies by resource type)
        embedded = response.get('_embedded', {})
        items = next(iter(embedded.values()), [])
        results.extend(items)
        if page >= response['pages']:
            break
        page += 1
    return results
```

---

## Default page size

The default `limit` is 5 items per page. Always specify an explicit `limit` in production code to avoid unintentional truncation of large result sets.

---

## Sorting and ordering

Some list endpoints support `order_by` and `sort` (or `sort_descending`) parameters. For example, when `domain_guid` or `domain_name` is specified on the mailboxes endpoint, results are automatically ordered by local part ascending. Pass `sort_descending=1` to reverse the order.

Consult the API documentation for the full list of supported filters and sort options on each endpoint.
