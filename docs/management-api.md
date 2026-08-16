> [← Documentation home](../README.md) · [Simple Auth](simple-auth.md) · [Bedrock](bedrock.md) · [Quicksilver](quicksilver.md) · [Server-side Variables](variables.md) · **Management API**

## Management API

The Management API lets a server-side tool manage one System Locker system over REST and JSON. Keep management credentials on your server. Never embed them in desktop software or publish them in a repository.

Version 2 is available at `https://systemlocker.net/api/v2`.

> Version 1 will be deactivated after September 30, 2026. Migrate existing integrations to v2 before then.

### Create a credential

Open the Systems page in the developer portal, select a system, and create a Management API v2 credential. Each credential belongs to that system and has only the scopes you select. Its complete value is shown once:

```text
slm_<token_id>_<secret>
```

Send it as a bearer credential:

```http
Authorization: Bearer slm_<token_id>_<secret>
Accept: application/json
```

Requests with a body must also send `Content-Type: application/json`. A credential cannot access another system, even when both systems have the same developer. Revoke a credential immediately if it may have been exposed.

Each credential can make 10 requests per 5 seconds. A rate-limited request returns HTTP 429 and a `Retry-After` header.

### Scopes

- `systems.read`, `systems.update`, `systems.delete`
- `keys.create`, `keys.read`, `keys.update`, `keys.delete`
- `variables.create`, `variables.read`, `variables.update`, `variables.delete`
- `security.read`

A missing scope returns HTTP 403. A resource outside the credential's system returns HTTP 404.

### Systems

| Method   | Path                             | Purpose                                                                                                                    |
| -------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `GET`    | `/api/v2/systems`                | List the credential's bound system.                                                                                        |
| `GET`    | `/api/v2/systems/{system}`       | Read its name, ID, version, program hash, and pause state.                                                                 |
| `GET`    | `/api/v2/systems/{system}/statistics` | Read online and total user counts for the system.                                                                      |
| `PATCH`  | `/api/v2/systems/{system}`       | Update `version` and/or `program_hash`. Send `null` or an empty string to clear the program hash.                          |
| `PUT`    | `/api/v2/systems/{system}/pause` | Pause authentication and end active sessions.                                                                              |
| `DELETE` | `/api/v2/systems/{system}/pause` | Resume authentication. An optional JSON body of `{"compensate": true}` extends active key expiries by the paused duration. |
| `DELETE` | `/api/v2/systems/{system}`       | Permanently delete the system and its associated data.                                                                     |

Statistics use the same user definition as v1: an account is counted once, while a redeemed key without an account is counted once by key. `online_users` is refreshed about every two minutes; `total_users` is refreshed hourly. Each value includes its own `*_computed_at` timestamp in RFC 3339 UTC.

### License keys

Create 1–100 keys with `POST /api/v2/systems/{system}/keys`:

```json
{
    "count": 2,
    "notes": "August order",
    "format": "@-%%%%-%%%%-%%%%",
    "free_trial": false,
    "expiry": {
        "type": "after_redemption",
        "seconds": 2592000
    }
}
```

`format` defaults to `@-%%%%-%%%%-%%%%`. Other custom formats require a plan with custom-key formatting and must contain exactly one `@` or `!` system-name placeholder plus 9–42 `%` random-character placeholders. `reseller` can contain a reseller token when the plan and selected system permit it.

Expiry is one of:

- `{"type": "perpetual"}`
- `{"type": "after_redemption", "seconds": 2592000}`
- `{"type": "at", "at": "2026-09-01T00:00:00Z"}`

Explicit timestamps can also be Unix seconds. RFC 3339 input must be UTC and end in `Z`. Timestamp output always uses RFC 3339 UTC.

| Method   | Path                                                    | Purpose                                                                        |
| -------- | ------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `GET`    | `/api/v2/systems/{system}/keys/{licenseKey}`            | Returns redemption status, HWID present, frozen, and all timestamps            |
| `PATCH`  | `/api/v2/systems/{system}/keys/{licenseKey}`            | Freeze or unfreeze with `{"frozen": true}`.                                    |
| `POST`   | `/api/v2/systems/{system}/keys/{licenseKey}/hwid-reset` | Reset one key's HWID.                                                          |
| `POST`   | `/api/v2/systems/{system}/keys/hwid-reset`              | Reset every HWID in the system.                                                |
| `POST`   | `/api/v2/systems/{system}/keys/{licenseKey}/time`       | Add time with a positive `seconds` integer. Perpetual keys cannot be extended. |
| `DELETE` | `/api/v2/systems/{system}/keys/{licenseKey}`            | Permanently delete one key.                                                    |

Adding time to an unredeemed duration-based key extends its future redemption duration without starting its clock.

### Server-side variables

| Method   | Path                                        | Purpose                                                                |
| -------- | ------------------------------------------- | ---------------------------------------------------------------------- |
| `POST`   | `/api/v2/systems/{system}/variables`        | Create a variable with `name`, `value`, and optional `protected`.      |
| `GET`    | `/api/v2/systems/{system}/variables/{name}` | Read one variable.                                                     |
| `PATCH`  | `/api/v2/systems/{system}/variables/{name}` | Update its `value`; optionally send a new `name` or `protected` value. |
| `DELETE` | `/api/v2/systems/{system}/variables/{name}` | Delete one variable.                                                   |

Variable names can contain letters, numbers, and underscores, up to 40 characters. Values can contain up to 500 characters. Your plan's per-system variable limit still applies.

### Security reads

`GET /api/v2/systems/{system}/security/ip-lookup?ip=8.8.8.8` performs an Aegis manual IP lookup when the developer's plan includes Aegis.

`GET /api/v2/systems/{system}/keys/{licenseKey}/logs` returns the latest five authentication logs for that key. Logging access is required. IP addresses are included only when the developer's logging level permits IP visibility.

### Errors

Errors use their HTTP status and one JSON shape:

```json
{
    "error": {
        "code": "INSUFFICIENT_SCOPE",
        "message": "This API key cannot create keys."
    }
}
```

Common statuses are 401 for an invalid credential, 403 for a missing scope or plan feature, 404 for an inaccessible resource, 422 for invalid input or a plan limit, 423 for a frozen or expired plan mutation, and 429 for rate limiting.

### Version 1 deactivation

Until September 30, 2026, version 1 accepts form-encoded POST requests at `/api/v1` using the legacy value stored in the system's `api_key` field. Its older `/api/endpoint2.php` path is also available during this migration period. Version 1 will be deactivated after that date. Version 2 credentials do not work with version 1, and legacy keys do not work with version 2.

For an existing v1 integration, every request includes `key`, the system's legacy API key.

Use `select` to read:

- `users` for the number of redeemed keys for the system.
- `key` for the redemption status of the key in `lkey`.
- `expiration` for the expiry date of the key in `lkey`.

Use `command` to perform an action:

- `hwidreset` resets the HWID for the key in `license`. Add `as_admin=false` to enforce the normal 30-day cooldown.
- `genkeys` creates one or more keys. Optional values are `expire` (0 through 5), `note` (up to 250 characters), and `count` (up to 100).
- `bankey` permanently deletes the key in `license`.
- `adjustexpiry` changes the expiry for the key in `license`. Send `newexpiry` and `tz`; set `newexpiry` to `0` for a permanent key.
- `systemhwidreset` resets the HWID for every key in the system.

V1 responses are legacy human-readable values. Continue using its established handling until you migrate to v2.