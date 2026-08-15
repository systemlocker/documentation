> [← Documentation home](../README.md) · [Simple Auth](simple-auth.md) · [Bedrock](bedrock.md) · [Quicksilver](quicksilver.md) · [Server-side Variables](variables.md) · **Management API**

## Management API

The Management API lets you build developer tools for key creation and hardware-ID resets. Send HTTP POST requests to `https://systemlocker.net/api/v1`. The legacy `https://systemlocker.net/api/endpoint2.php` path remains available for existing integrations.

This API can create and modify customer access. Keep its API key private and restrict any bot or internal tool that can call it.

### Request body

Every request requires `key`, your system's API key. It is different from the system ID and can be found or regenerated on the system settings page.

Use `select` to query data:

- `users` returns the number of redeemed keys for the system.
- `key` returns the redemption status of the key in `lkey`.
- `expiration` returns the expiry date of the key in `lkey`.

Use `command` to perform an action:

- `hwidreset` resets the HWID for the key in `license`. Add `as_admin=false` to enforce the normal 30-day cooldown.
- `genkeys` creates one or more keys. Optional values are `expire` (0 through 5), `note` (up to 250 characters), and `count` (up to 100).
- `bankey` permanently deletes the key in `license`.
- `adjustexpiry` changes the expiry for the key in `license`. Send `newexpiry` and `tz`; set `newexpiry` to `0` for a permanent key.
- `systemhwidreset` resets the HWID for every key in the system.

### Response body

A successful request returns HTTP 200 without the `error: true` response header. Responses are human-readable; display a clear result in your own developer tool and treat all other responses as failures.
