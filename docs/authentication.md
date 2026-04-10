# Authentication

All requests to the SYNAQ API must be authenticated using SWSS (SYNAQ Web Service Security) authentication, injected into HTTP headers. These three headers are required on every request:

```
synaq-api-user: {reseller API username}
synaq-api-salt: {current UNIX timestamp}
synaq-api-hash: {SHA1 hash of username + API key + salt}
```

Your username and API key are provisioned by SYNAQ. The salt must be a UNIX timestamp, and the hash is computed by concatenating the username, API key, and salt — in that order — and hashing the result with SHA1.

> **Replay protection:** Timestamps must not be reused. If your client makes more than one request per second, use a sub-second timestamp (milliseconds or microseconds) to ensure uniqueness.

> **Case sensitivity:** Some legacy endpoints return a lowercase `location` header instead of the standard `Location`. HTTP clients that treat headers case-sensitively must check for both forms whenever a `202 Accepted` is received.

---

## Examples

### PHP

```php
$username = 'client@example.com';
$apiKey   = 'your-api-key-from-synaq';
$salt     = microtime(true) * 10000;   // sub-second precision
$hash     = sha1($username . $apiKey . $salt);

$headers = [
    'synaq-api-user: ' . $username,
    'synaq-api-salt: ' . $salt,
    'synaq-api-hash: ' . $hash,
    'Content-Type: application/json',
    'Accept: application/json',
];
```

### Python

```python
import hashlib
import time

username = 'client@example.com'
api_key  = 'your-api-key-from-synaq'
salt     = str(int(time.time() * 10000))   # sub-second precision
hash_val = hashlib.sha1((username + api_key + salt).encode()).hexdigest()

headers = {
    'synaq-api-user': username,
    'synaq-api-salt': salt,
    'synaq-api-hash': hash_val,
    'Content-Type':   'application/json',
    'Accept':         'application/json',
}
```

### Node.js

```javascript
const crypto = require('crypto');

const username = 'client@example.com';
const apiKey   = 'your-api-key-from-synaq';
const salt     = String(Date.now());        // millisecond precision
const hash     = crypto
  .createHash('sha1')
  .update(username + apiKey + salt)
  .digest('hex');

const headers = {
  'synaq-api-user': username,
  'synaq-api-salt': salt,
  'synaq-api-hash': hash,
  'Content-Type':   'application/json',
  'Accept':         'application/json',
};
```

---

## Credentials and environments

SYNAQ provides separate credentials for staging and production:

- **Staging** — for development and testing; credentials provided by SYNAQ
- **Production** — for live traffic; credentials issued after successful staging integration

New integrators receive staging credentials first. Production credentials are issued once the integration is stable.

You will also receive your **reseller GUID** — the root-level identifier for all OU and package operations. Keep this and your API key secure; they grant full control over all entities under your reseller.
