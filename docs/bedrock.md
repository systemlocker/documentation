> [← Documentation home](../README.md) · [Simple Auth](simple-auth.md) · **Bedrock** · [Quicksilver](quicksilver.md) · [Server-side Variables](variables.md) · [Management API](management-api.md)

## Bedrock

Bedrock is the recommended production authentication API for all new integrations. Supporting account-based and key-only authentication, cryptographically-signed responses, and endless sessions, Bedrock is our most secure offering yet. It requires a paid developer plan with production access.

The Bedrock C++ reference implementation is the fastest way to get started. It includes a C++20 client, integration instructions, and full support for server-side variables and Invisible Folder-backed downloads. Embed the library's source in your project when integrating Bedrock; this keeps the client bundled with the rest of your code, allowing the strongest obfuscation and security layers to be applied.

Use POST https://systemlocker.net/auth/bedrock/init to initialize a session, then POST https://systemlocker.net/auth/bedrock/beat for heartbeats.

### Initialization request

Send either `username` and `password` for account authentication, or `key` for key-only authentication. Do not send both forms of credentials together.

`system` - Your 20-character system ID.

`hwid` - The machine identifier for this customer.

`version` - Required when your system has a version configured. Send `bypass` only when version checking is not needed.

`beatrate` - Optional heartbeat interval in seconds. The default is 30; valid values are 25 through 3600.

`digest` - Required when the system has a Program Hash configured. It must exactly match the configured value.

`challenge` - A fresh client-generated random string, 64 through 100 characters long. Generate a new challenge for every request.

`init-if` - Optional boolean. Send `true` to create a short-lived Invisible Folder token for the successful request.

`variables` - Optional array of server-side variable names. On an authenticated initialization, Bedrock returns a map of the requested names; unavailable names map to `false`.

`X-Bedrock-Key-Id` - Optional request header identifying the active signing key the client expects. Omit it to use the system's current active key.

### Initialization response

The response body is a base64url transport value containing an Ed25519 signature followed by the exact JSON payload. Verify the signature with the public key you distribute with your application before reading or trusting the JSON. TLS still provides confidentiality.

Successful responses have `response_code` `OK` or `OUTDATED`, include `authed: true`, and include `session_token`. The response also echoes `challenge`, identifies the system, and includes the server time. Key-only requests include `license_key_hash`; account requests include `username_hash`.

When requested, successful responses also include `invisible_folder_token` and the `variables` map. Full usernames and license keys are never returned.

Check `response_code` rather than assuming any signed response authorizes access. Common values include `INVALID_CREDENTIALS`, `INVALID_KEY`, `HWID_MISMATCH`, `EXPIRED_KEY`, `PROGRAM_DIGEST_MISMATCH`, and `PRODUCTION_AUTH_UNAVAILABLE`.

### Heartbeat request and response

Send `session_token`, `system`, and a fresh `challenge` to `POST https://systemlocker.net/auth/bedrock/beat` at the accepted heartbeat interval. Use the replacement `session_token` from every successful response for the next heartbeat.

Set `init-if` to `true` to receive a new short-lived `invisible_folder_token` in a successful heartbeat response.

If a response is lost, repeat the immediately previous token with the same challenge during the next heartbeat interval. Bedrock returns the exact cached signed response once; changing the challenge or waiting too long invalidates that retry.

Terminal responses include a `termination_message`. Stop access when the response code reports a terminated, stale, early, revoked, or otherwise invalid session.

### Downloads and auto-updates

Make sure to use the initialization endpoint with `init-if=true` if you're planning to download files protected by the System Locker - Advanced permission, which is what we recommend for maximum security. This will return a token that's valid for 10 minutes. To continue downloading files after that token expires, simply add `init-if=true` to the next heartbeat request.

The reference C++ implementation of Bedrock includes functions for downloading files; `downloadIfNew()` is designed specifically for an autoupdating feature: it checks to see if the file you're requesting is neweer than the last version before performing a download.
