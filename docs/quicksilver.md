> [← Documentation home](../README.md) · [Simple Auth](simple-auth.md) · [Bedrock](bedrock.md) · **Quicksilver** · [Server-side Variables](variables.md) · [Management API](management-api.md)

## Quicksilver

Quicksilver is a production session-authentication API for existing integrations. It authenticates once and then uses lightweight heartbeats while the program is running. It supports account-based and key-only authentication and requires a paid developer plan with production access.

Quicksilver is planned to be phased out during 2027. Existing integrations can continue to use it for now; new production integrations should use [Bedrock](bedrock.md).

The complete [Quicksilver PDF](https://systemlocker.net/documentation/downloads/quicksilver.pdf) is also available.

### Initial request

For account authentication, send `POST https://systemlocker.net/quicksilver/init`. For key-only authentication, use `POST https://systemlocker.net/quicksilver/init-mikros`.

`username` and `password` - Required for account authentication.

`key` - Required for key-only authentication instead of account credentials.

`system` - Your 20-character system ID.

`hwid` - A machine identifier used for your hardware-locking rules.

`version` - Optional unless the system has a configured version. Send `bypass` when version checking is not needed.

`beatrate` - Optional heartbeat interval in seconds. The default is 30; valid values are 25 through 3600.

`digest` - Required when the system has a Program Hash configured. It must exactly match the configured value.

`init-if` - Optional boolean. Send `true` to append a short-lived Invisible Folder token to a successful initialization response.

An unsuccessful request returns a plain-text value such as `bad u/p`, `bad key`, `paused`, `frozen`, `hwid banned`, `expired key`, `no production`, `outdated`, or `digest`. Treat any unexpected value as a failure.

A successful response is `token|identifier|timestamp:hash`. The first field begins with `TT`; confirm that `identifier` matches the username or key you submitted, validate the timestamp, and verify the SHA-1 hash over the part before the colon.

### Heartbeat request

Send `POST https://systemlocker.net/quicksilver/beat` at the exact accepted interval.

`token` - The token from initialization or the prior successful heartbeat.

`system` - The system ID used to create the session.

A response beginning with `TTr` is the replacement token and the only successful heartbeat response. Store it for the next heartbeat. Other responses include `bad request`, `bad session token`, `stale token`, `rate limit`, `expired key`, or `terminated [reason]`.
