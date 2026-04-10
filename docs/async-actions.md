# Asynchronous Actions

Any API call that triggers backend provisioning work returns `202 Accepted` instead of an immediate result. The response includes a `Location` header pointing to the action resource you should poll.

---

## Identifying async responses

```
202 Accepted
Location: /api/v1/domains/{domain-guid}/actions/{action-id}
```

> **Header case:** Some legacy endpoints return a lowercase `location` header. Clients that treat HTTP headers case-sensitively must check for both `Location` and `location` whenever they receive `202 Accepted`.

---

## Polling an action

Send `GET` requests to the action URL until you receive a terminal state:

```
GET /api/v1/domains/{domain-guid}/actions/{action-id}.json
```

### Pending (still in progress)

```json
{
  "state": "pending"
}
```

### Finished (success)

```json
{
  "id": 1234,
  "action": "provision",
  "state": "finished",
  "errors": [],
  "exclusions": []
}
```

### Error (failed)

```json
{
  "id": 1234,
  "action": "provision",
  "state": "error",
  "errors": [
    "A human-readable description of the failure",
    "Potentially multiple messages"
  ]
}
```

---

## Recommended polling strategy

Poll at a short interval initially, backing off if the action is still pending after several attempts. Most actions complete within a few seconds; Archive-related actions may take longer.

```php
function pollAction(string $baseUrl, string $actionPath, array $headers, int $maxAttempts = 60, int $interval = 3): array {
    $url = $baseUrl . $actionPath;
    for ($attempt = 0; $attempt < $maxAttempts; $attempt++) {
        $response = httpGet($url, $headers); // use your HTTP client of choice
        if ($response['status'] === 404) {
            // Special case: mailbox DELETE removes the action record on completion
            return ['state' => 'finished'];
        }
        $data = $response['body'];
        if ($data['state'] === 'finished') return $data;
        if ($data['state'] === 'error') throw new RuntimeException('Action failed: ' . implode(', ', $data['errors']));
        sleep($interval);
    }
    throw new RuntimeException("Action did not complete after {$maxAttempts} attempts");
}
```

```python
import time
import requests

def poll_action(base_url, action_path, headers, max_attempts=60, interval=3):
    url = base_url + action_path
    for attempt in range(max_attempts):
        response = requests.get(url, headers=headers)
        if response.status_code == 404:
            # Special case: mailbox DELETE removes the action record on completion
            return {'state': 'finished'}
        data = response.json()
        state = data.get('state')
        if state == 'finished':
            return data
        if state == 'error':
            raise Exception(f"Action failed: {data.get('errors')}")
        time.sleep(interval)
    raise TimeoutError(f"Action did not complete after {max_attempts} attempts")
```

```javascript
async function pollAction(baseUrl, actionPath, headers, maxAttempts = 60, interval = 3000) {
    const url = baseUrl + actionPath;
    for (let attempt = 0; attempt < maxAttempts; attempt++) {
        const res = await fetch(url, { headers });
        if (res.status === 404) {
            // Special case: mailbox DELETE removes the action record on completion
            return { state: 'finished' };
        }
        const data = await res.json();
        if (data.state === 'finished') return data;
        if (data.state === 'error') throw new Error(`Action failed: ${data.errors}`);
        await new Promise(resolve => setTimeout(resolve, interval));
    }
    throw new Error(`Action did not complete after ${maxAttempts} attempts`);
}
```

---

## Special case: mailbox deletion

The `DELETE /api/v1/mailboxes/{guid}.json` endpoint automatically creates a delete action. Unlike other actions, **a successful mailbox delete removes the action record itself**, so polling will return `404 Not Found` when the deletion is complete.

Treat a `404` response on a delete action poll as a `finished` state.

---

## Error handling

If an action reaches the `error` state:

1. Record the error messages from the `errors` array — these can be passed to SYNAQ support for diagnosis.
2. Fix the underlying issue (e.g. incorrect service field values, a resource conflict on the backend).
3. Submit the same action again — failed actions are never retried automatically.

---

## Package actions

Actions on packages follow exactly the same pattern, but use the package actions endpoint:

```
POST /api/v1/packages/{package-guid}/actions.json
GET  /api/v1/packages/{package-guid}/actions/{action-id}.json
```
