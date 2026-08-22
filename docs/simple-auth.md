> [← Documentation home](../README.md) · **Simple Auth** · [Bedrock](bedrock.md) · [Quicksilver](quicksilver.md) · [Server-side Variables](variables.md) · [Management API](management-api.md)

## Simple Auth

Simple Auth provides a direct answer to each authentication request. Choose Goliath when customers sign in with a System Locker account, or Mikros when they enter a license key. Both endpoints accept HTTP POST requests over HTTPS.

### Goliath

Goliath is the default account-based API. Send requests to `https://systemlocker.net/auth/goliath`.

#### Request body

`system` - Your 20-character system ID.

`username` - The customer's System Locker username.

`password` - The customer's System Locker password. For a Google account, send the system-specific password created through the Google SSO flow below.

`hwid` - A machine identifier used for your hardware-locking rules. If you do not want hardware locking, send a consistent application value such as `1`.

`version` - Optional unless the system has a version configured. It must then match that configured version.

`digest` - Required when the system has a Program Hash configured. Send the exact configured value; omit it only when no Program Hash is configured.

#### Response body

`true` means the customer is authenticated and can access the program. Treat every other response as a failure and handle these documented values deliberately:

`false` - A required value is missing.

`dbe` - The service could not complete the request.

`no username`, `no password`, `no sys`, or `no hwid` - The named request field is missing.

`paused` - The developer paused this system.

`not verified` - The customer's account is not verified.

`bad u/p` - The supplied account credentials are not valid.

`bad keys` - The account has no valid key for this system.

`frozen`, `banned`, `spoofsuspected`, `user limit`, `expired key`, or `hwid` - The customer's access or machine does not meet the system's requirements.

`outdated` - The submitted version does not match the configured system version.

`digest` - The Program Hash is missing or does not match.

### Google SSO account password

Google accounts use a system-specific password instead of a normal System Locker password. Your client must recognize these Google SSO responses before treating a plain-text response as an ordinary authentication failure:

`sso [LINK]` - No password has been created for this account and system.

`ssoexp [LINK]` - The previous system password has expired.

`ssowrong [LINK]` - An active system password exists, but the supplied password does not match it.

In each case, extract the complete URL after the first space and let the customer open it. After they sign in with Google, the page displays a password for that system. Submit that password as `password` with the same `username` and `system`, then retry the original request. The password expires after 180 days, works only for the system named in the link, and is replaced when the customer completes Google sign-in again. Do not reuse it for another system or write it to logs.

### Mikros

Mikros is the license-key API and is available on every plan. Send requests to `https://systemlocker.net/auth/mikros`.

#### Request body

`system` - Your 20-character system ID.

`key` - The customer's license key.

`hwid` - A machine identifier used for your hardware-locking rules. Treat this as application data, not as a password. If you do not want hardware locking, send a consistent application value such as `1`.

`version` - Optional unless the system has a version configured. It must then match that configured version.

`digest` - Required when the system has a Program Hash configured. Send the exact configured value; omit it only when no Program Hash is configured.

#### Response body

`true` means the key is valid and the customer can access the program.

`false` - A required value is missing.

`dbe` - The service could not complete the request.

`no key`, `no sys`, or `no hwid` - The named request field is missing.

`paused`, `bad key`, `frozen`, `banned`, `spoofsuspected`, `destitute`, `user limit`, `expired key`, `hwid`, `outdated`, or `digest` - The request could not be authorized. Display a useful customer-facing message where appropriate, and treat unknown responses as a failure.
