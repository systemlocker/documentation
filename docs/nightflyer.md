> [← Documentation home](../README.md) · [Simple API](simple-auth.md) · [Bedrock](bedrock.md) · **Nightflyer** · [Management API](management-api.md) · [Server-side Variables](variables.md) · [SL-HWID](sl-hwid.md) · [Quicksilver](quicksilver.md)

## Nightflyer

Nightflyer is our offline authorization API for desktop software. It lets licensed customers keep using your application with no internet connection — through outages, travel, and restricted networks — while every authorization stays signed, machine-bound, and time-limited. Each installation identifies itself with a device-held signing key and receives a signed offline lease it must renew before that lease expires. A copied lease will not run on another machine, tampered fields invalidate the signature, and you choose exactly how long offline access can last: anywhere from 20 minutes to 90 days.

Nightflyer complements the Simple API and Bedrock. Use the Simple API when the licensing check runs on a trusted central server, Bedrock when connected software runs on user-controlled hardware, and Nightflyer when the customer application must keep working without a connection. Nightflyer requires a paid developer plan with the API enabled. Enable it per system and set the maximum offline window in System Management; the license policy you already know from Bedrock — application version, program digest, hardware bans, and expiry — is applied when a lease is issued and again at every renewal.

Use POST https://systemlocker.net/api/nightflyer/v1/authorize to obtain a lease, POST https://systemlocker.net/api/nightflyer/v1/renew to extend it, POST https://systemlocker.net/api/nightflyer/v1/status to pick up signed revocation state, and POST https://systemlocker.net/api/nightflyer/v1/end to release a lease early.

### Offline leases

A lease is a compact, Ed25519-signed token with an absolute expiry. The server calculates every timestamp from its own clock; nothing the client reports can lengthen a lease. Expiry is capped by your system's maximum offline setting, the duration the client requested, and by the license's own expiry, so a lease never outlives its license. Lowering the maximum only affects leases issued afterwards, never one an offline device already holds.

Nightflyer signs with the same per-system Ed25519 keys as Bedrock, but with its own token types, which can never be confused with a Bedrock response.

There are two limitations that this system accepts, in order to offer truly offline-enabled auth. First, System Locker cannot revoke a device it cannot reach: the maximum offline window is also the longest a revocation can be delayed, and enforcement happens the moment the device reconnects. Second, Nightflyer raises the cost of cracking attempts, but it cannot prevent a determined attacker from patching your application; always apply the same obfuscation layers you already use.

### Installation keys and request proofs

Generate a unique P-256 signing key for each installation of your software and protect it with the platform's key store, such as a TPM, Secure Enclave, or OS keychain. System Locker identifies the installation by the RFC 7638 thumbprint of the key's public JWK, and every request must prove possession of the matching private key.

Need to check a public JWK thumbprint while integrating? Use the browser-local [JWK Thumbprint Calculator](https://systemlocker.net/tools/jwk-thumbprint). Never paste a private key or credential into it.

All four endpoints accept only POST requests with `Content-Type: application/json`. The body must be UTF-8 JSON of at most 16 KB; duplicate member names, unknown fields, and trailing data are rejected.

Every request carries a compact JWS in the `Nightflyer-Proof` header, signed with the installation key. Its protected header contains exactly `alg` (always `ES256`), `typ` (always `nightflyer-proof+jwt`), and `jwk` with the four public members `kty`, `crv`, `x`, and `y`. Its payload contains exactly these claims:

`nightflyer_version` - Integer; currently always `1`.

`htm` - Always `POST`.

`htu` - The exact endpoint URI, such as https://systemlocker.net/api/nightflyer/v1/authorize.

`system` - Your 20-character system ID.

`iat` - Integer Unix time, within five minutes of server time.

`jti` - The unpadded base64url encoding of 16 fresh random bytes. A proof may be submitted once; reusing a `jti` is rejected with `PROOF_REPLAYED`.

`rqh` - A base64url SHA-256 hash over the RFC 8785 canonicalization of the request's non-secret fields (the fields documented for each endpoint below). Credentials and the lease itself are never hashed into the proof.

`ath` - On renew, status, and end only: a base64url SHA-256 hash of the compact lease presented in the body.

If a response is lost, retry the exact same request bytes within a short window and the server returns the cached response. Changing the request requires a fresh proof.

### Authorization request

Send either `license_key` for key authentication, or `username` and `password` for account authentication. Do not send both forms of credentials together. Google accounts use a system-specific password created through Google SSO, not a normal account password.

`system` - Your 20-character system ID.

`identity_type` - Either `key` or `account`.

`version` - Required when your system has a version configured; otherwise send an empty string. Send `bypass` only when version checking is not needed.

`digest` - Required when the system has a Program Hash configured. It must exactly match the configured value; send an empty string when there is none.

`sl_hwid` - The machine identifier for this customer.

`storage_mode` - Either `memory` or `persistent`, telling the server how your client keeps the lease.

`requested_offline_seconds` - The offline duration to request, from 1200 through 7,776,000. The server issues the shorter of this and your system's maximum.

`sdk_version` - A string identifying your integration, up to 64 characters.

`protocol_version` - Integer; currently always `1`.

`trusted_kids` - The key IDs of the signing public keys pinned into your application, in ascending order with no duplicates, at most 32 entries.

`keyset_sequence` and `status_sequence` - The highest sequence values accepted from signed responses so far; send `0` when you have none. In protocol version 1 these reports are advisory: they do not interrupt a request or change server recovery state.

Clients must still reject signed responses that move either sequence backward. System Locker uses independently authenticated server recovery records rather than client-reported v1 counters when restoring sequence state.

Nightflyer uses [SL-HWID](https://github.com/systemlocker/SL-HWID), our fault-tolerant hardware identifier, as its machine binding. Only a hash of the value is ever signed into a lease; the raw identifier never leaves the request.

### Successful responses

The response body is ordinary JSON framing — `protocol_version`, `request_jti`, and, as applicable, `lease`, `status`, `key_transitions`, and `decision`. Every security decision lives inside a signed object; the framing itself never authorizes access. Verify the signatures before trusting anything.

The lease is a compact JWS with `typ: nightflyer-auth+jwt` and `alg: EdDSA`. Its claims identify the system (`aud`), an opaque license subject (`sub`), the authorization (`jti`), the installation (`device_id` and the `cnf.jkt` thumbprint of your installation key), a hash of the SL-HWID value, the `generation`, `iat`, `nbf`, `exp`, `renew_after`, a `policy_revision`, and the `request_jti` of your proof, so a response cannot be substituted between requests. `entitlements` is reserved and always an empty object in protocol version 1.

The signed status manifest, `typ: nightflyer-status+jwt`, reports the current `grant_status` (`active`, `reclaim_pending`, `ended`, or `revoked`), a `status_sequence` that only increases, the lowest acceptable `minimum_generation`, revoked signing key IDs, and a `next_update` time. Reject a manifest with a sequence lower than one already accepted, and reject any lease whose generation is below the manifest's minimum.

When the active signing key is not among your pinned `trusted_kids`, `key_transitions` carries the shortest chain of `nightflyer-keyset+jwt` objects leading from a key you already trust to the active key. Add a new key only after verifying its transition with an already pinned key; never trust a public key merely because it arrived in a response.

When verifying signed objects, hard-code `EdDSA` as the only server algorithm, require the exact token type, check the issuer and audience, reject duplicate JSON members and unknown protocol versions, and verify the signature over the exact received bytes rather than re-serializing first. Confirm the installation thumbprint, SL-HWID hash, and `request_jti` match the request you sent, and treat `exp` as absolute.

### Renewal

Wait until `renew_after`, then send the lease plus the same non-credential binding fields as during authorization: `protocol_version`, `sdk_version`, `system`, `version`, `digest`, `sl_hwid`, `trusted_kids`, `keyset_sequence`, and `status_sequence`.

A successful renewal returns a new lease with an incremented `generation` and a freshly computed `exp`, still capped by your system's maximum and the license expiry. The new `renew_after` is about a third of the lease length, between 5 minutes and 24 hours, with per-device jitter so fleets do not synchronize. Renewing early returns the current lease with a signed `NOT_DUE` decision rather than extending it.

If another process renewed first, renewal fails with `STALE_GENERATION`; reload and verify the shared state instead of retrying in a loop. Renewing an expired lease fails with `LEASE_RECLAIM_PENDING`, which is a strong sign to send `/end` to release the lease. Exact retries follow the same cached-response rule as every other operation.

### Status synchronization

A status request presents the lease with `protocol_version`, `sdk_version`, `system`, `trusted_kids`, `keyset_sequence`, and `status_sequence`. It returns the latest signed status manifest without extending the lease or changing its generation, so a reconnecting device learns immediately about unbinding events, signing-key revocations, or policy changes. It accepts an expired lease, which lets a device that resurfaced after expiry discover its grant's terminal state and release the server-side reservation through an end request.

### Ending a lease

Call the end endpoint deliberately — when the customer signs out, switches license, or otherwise gives up offline access — not on every application close (unless that is your desired end state, and you expect the user to come back online before opening the application again). Include `end_reason`, a non-empty string of up to 128 characters. Ending a lease marks the grant terminal, records the revocation, and immediately releases this installation's server-side reservation, so a new lease or another authentication API can be used right away. The response is a signed `LEASE_ENDED` decision (`LEASE_REVOKED` if the lease was already revoked), and it states whether this request released the reservation. An exact retry is idempotent.

A device that is already offline keeps its old lease until `exp`; ending a lease cannot reach it. The end endpoint also accepts a recognized expired lease, so a device that reconnects after expiry can release its server-side reservation.

### Exclusivity with other authentication APIs

For paid keys, Nightflyer is mutually exclusive with the Simple API and Bedrock for the same license and system. While any paid Nightflyer lease is active, those APIs deny authentication with `NIGHTFLYER_LEASE_ACTIVE`; conversely, authorization fails with `AUTH_MODE_IN_USE` while a paid Bedrock or Quicksilver session is live, and with `LEASE_ALREADY_ACTIVE` if this installation already holds a lease. A license may hold leases for several installations when each passes your HWID policy, so treat this as machine licensing: watch the lease list in System Management and unbind anything unusual.

After a lease expires without being ended, its reservation remains for a reclaim window — currently 24 hours — during which authorization returns `LEASE_RECLAIM_PENDING`. You can release it sooner at any time through Unbind Lease in System Management or the Management API. Unbinding invalidates the named lease and releases its machine claim; it does not ban the key or installation, so the first valid attempt to authorize afterward receives the new binding.

Free-trial keys are exempt from these rules: they may hold any number of concurrent leases, they do not block other authentication APIs, and they never claim or update a license's HWID.

### Errors and response codes

A recognized denial returns an appropriate HTTP status — 401, 403, or 409 — with the standard framing and a signed decision, `typ: nightflyer-decision+jwt`, carrying the `response_code`, your `request_jti`, a `lease_disposition` of `unchanged` or `revoked`, and, where relevant, whether the denial released the server-side reservation. Common values include `INVALID_KEY`, `INVALID_CREDENTIALS`, `HWID_MISMATCH`, `HWID_BANNED`, `EXPIRED_KEY`, `KEY_FROZEN`, `OUTDATED`, `PROGRAM_DIGEST_MISMATCH`, `PRODUCTION_AUTH_UNAVAILABLE`, and `NIGHTFLYER_DISABLED`.

Transport and request failures return RFC 9457 Problem Details with a `code` member instead. An unsigned response never authorizes access; treat it as a failed online attempt.

If a newer protocol version is ever required, recognizable clients receive HTTP 426 with a signed `PROTOCOL_UPGRADE_REQUIRED` decision containing update details and whether the cached lease remains valid until its existing expiry. Versioned endpoints stay backward compatible; a breaking change ships as a new version alongside the old one.

Temporary failures return 503, and rate-limited requests return 429 with `Retry-After`. A network failure alone never ends a still-valid lease: keep authorizing offline until `exp` and retry with backoff.
