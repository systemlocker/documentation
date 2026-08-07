## Introduction

This article describes how to use each of the available System Locker authentication/authorization API endpoints, as well as the SSV and management APIs. Documentation for our data and action API features is not yet available on this page. Please contact support for more information.

**Authentication API**

- [Goliath (default; account-based)](#goliath)
- [Mikros (key-based)](#mikros)
- [Quicksilver (production session auth)](#quicksilver)

**Variables API**

- [Unauthenticated Server Side Variable](#variable)

**Management API**

- [Key Management API](#management)

Every request to System Locker APIs must be an [HTTP POST request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST). Body parameters vary for each endpoint and depending on your specific goal.

### Goliath

Goliath is the default API for authenticating at System Locker. It can be reached at `https://systemlocker.net/auth/goliath`

#### Request body

In order to authenticate, the following pieces of data must be provided:

`system` - This is the system ID. It's a 20 character alphanumeric string. You can find it on any page where your system is listed.

`username` - The user's username. They can choose one when [signing up](/signup).

`password` - The user's password.

`hwid` - An identifier used to determine if the user is running the program on a second device. **If you do not wish to have this checked, best practice is to hard code a value for all users, such as `1`.**

`version` - Optional if the system does not have a version. Otherwise, it must match the value set on the [system setting page for that system](/devs/sysmanagement).

`digest` - Required when the system has a Program Hash configured. Send the exact configured value with every authentication request; omit it only when no Program Hash is configured.

#### Response body

There are a number of possible responses. One of these values will be in the response body:

`false` - A required peice of information is missing.

`dbe` - Something failed on the server.

`no username` - The `username` field was not present in the request.

`no password` - The `password` field was not present in the request.

`no sys` - There was no system provided in the request.

`no hwid` - There was no hwid provided in the request.

`paused` - The system has been paused by its developer. Users should be told to try again later.

`not verified` - The user has not yet verified the email address for their account.

`bad u/p` - The username or password did not match any in our database.

`bad keys` - The user does not have a valid key for your system.

`frozen` - The user's key has been frozen, or your developer plan has been frozen. Manual action by the developer, a reseller, or System Locker support is required.

`banned` - Aegis IP Intelligence detected the user and the developer selected Ban or Block HWID for that detection.

`spoofsuspected` - The user's HWID matched our list of known-malicious hardware identifiers. Their key has been frozen and they should not be allowed to access your system.

`user limit` - Your plan's active-user limit has been reached. The new user's key cannot be redeemed until the limit is increased.

`expired key` - The user's key is expired and they should not be allowed to access your system.

`hwid` - The hardware identifier provided does not match the one in our database. They should have it reset if they are on a new PC.

`outdated` - The provided version does not match the listed version for the system. They should check for updates.

`digest` - The Program Hash is missing or does not match the value configured for the system.

The next few responses are specifically for users who signed up with a Google account. These must be supported.

`sso [LINK]` - The user should be prompted to visit the link in this response. It is separated by a space, so check if the response _starts with_ `sso `.

For example, `sso https://systemlocker.net/user/sso?system=...`

`ssoexp [LINK]` - Similar to the above response, except their previous sign-in token has expired. The user should be prompted to visit the link in this response.
Tokens expire regularly. This is expected behavior.

`ssowrong [LINK]` - Similar to the above response, except the provided token is wrong. The user should be prompted to visit the link in this response.

**Google account users must follow the link. After signing in on the website, they will get a token. This will serve as their password until it expires. You should save this so they don't have to sign in every time they run your program.**

`true` - The user is who they claim to be, and they can access your program. Every check has succeeded.

It is recommended to use a `switch` or `if else` series to check for each of these responses. For any unlisted or unexpected response, tell the user to contact the developer of the program (that's you!).

### Mikros

Mikros is the key-based auth API for System Locker and is available on every plan. It can be reached at `https://systemlocker.net/auth/mikros`

There is a [PDF version](/downloads/mikros.pdf) of Mikros documentation available as well. It may be easier to understand.

#### Request body

In order to authenticate, the following pieces of data must be provided:

`system` - This is the system ID. It's a 20 character alphanumeric string. You can find it on any page where your system is listed.

`key` - The user's license key. This is what you will provide them with when they purchase access to your system.

`hwid` - An identifier used to determine if the user is running the program on a second device. For the Mikros API, this is comparable to a password. Hardware identifiers are stored in _plaintext_ so do not put unhashed passwords in this field! **If you do not wish to have this checked, best practice is to hard code a value for all users, such as `1`.**

`version` - Optional if the system does not have a version. Otherwise, it must match the value set on the [system setting page for that system](/devs/sysmanagement).

`digest` - Required when the system has a Program Hash configured. Send the exact configured value with every authentication request; omit it only when no Program Hash is configured.

#### Response body

There are a number of possible responses. One of these values will be in the response body:

`false` - A required peice of information is missing.

`dbe` - Something failed on the server.

`no key` - The `key` field was not present in the request.

`no sys` - There was no system provided in the request.

`no hwid` - There was no hwid provided in the request.

`paused` - The system has been paused by its developer. Users should be told to try again later.

`bad key` - The provided key does not exist or does not belong to this system.

`frozen` - The user's key has been frozen. This is the result of manual action by the developer or a reseller who has sufficient permissions.

`banned` - Aegis IP Intelligence detected the user and the developer selected Ban or Block HWID for that detection.

`spoofsuspected` - The user's HWID matched our list of known-malicious hardware identifiers. Their key has been frozen and they should not be allowed to access your system.

`destitute` - Your developer account's plan is frozen or expired. Contact System Locker support immediately.

`user limit` - Your plan's active-user limit has been reached. The new user's key cannot be redeemed until the limit is increased.

`expired key` - The user's key is expired and they should not be allowed to access your system.

`hwid` - The hardware identifier provided does not match the one in our database. They should have it reset if they are on a new PC.

`outdated` - The provided version does not match the listed version for the system. They should check for updates.

`digest` - The Program Hash is missing or does not match the value configured for the system.

`true` - The user is who they claim to be, and they can access your program. Every check has succeeded.

It is recommended to use a `switch` or `if else` series to check for each of these responses. You may not want to show the user anything specific for the `destitute` response. In that case, or in the case of any unlisted response, tell them to contact the developer of the program (that's you!).

### Quicksilver

Quicksilver is the paid production session-auth API for System Locker. It authenticates the user once, then uses lightweight heartbeat requests for the rest of the program's lifetime. It supports both account-based and key-only authentication. Both initialization flows require a paid developer plan with production access.

The complete integration guide is also available in the [Quicksilver PDF](/downloads/quicksilver.pdf).

#### Initial request

For account-based authentication, send the initial request to `https://systemlocker.net/quicksilver/init`. For key-only authentication, we recommend using `https://systemlocker.net/quicksilver/init-mikros` instead. Send this request once when the program starts; use the heartbeat endpoint for subsequent checks.

##### Request body

`username` - Required for account-based authentication. The user's System Locker username.

`password` - Required for account-based authentication. The user's System Locker password.

`key` - Required for key-only authentication instead of `username` and `password`. The user's license key.

`system` - This is the system ID. It's a 20 character alphanumeric string. You can find it on any page where your system is listed.

`hwid` - A unique identifier of the machine. **If you do not wish to have this checked, best practice is to hard code a value for all users, such as `1`.**

`version` - Optional if the system does not have a version. Otherwise, it must match the version configured for the system. Send `bypass` to skip this check.

`beatrate` - Optional. The requested interval, in seconds, between heartbeat requests. The default is 30 seconds; valid values are from 25 through 3600.

`digest` - Required when the system has a Program Hash configured. Send the exact configured value with every initialization request; omit it only when no Program Hash is configured. Heartbeat requests do not include a digest.

##### Response body

An unsuccessful initial request returns one of the following plain-text values:

`false` - A required piece of information is missing, such as an empty `hwid`.

`no u/p` - The `username` or `password` field was not present for an account-based request.

`no key` - The `key` field was not present for a key-only request.

`no sys` - There was no system provided in the request.

`dbe` - Something failed on the server.

`not verified` - The account's email or Discord account has not been verified.

`bad u/p` - The username or password did not match an account.

`bad key` - The supplied key does not exist or does not belong to this system.

`bad keys` - The account does not have a valid key for this system.

`paused` - The system has been paused by its developer.

`frozen` - The user's key has been frozen.

`banned` - Aegis IP Intelligence detected the user and the developer selected Ban or Block HWID for that detection.

`hwid banned` - The supplied HWID is on the developer's HWID ban list.

`spoofsuspected` - The HWID is known to be malicious. The associated key has been frozen.

`destitute` - The developer account's plan is frozen or expired.

`no production` - The developer's plan does not include production access.

`user limit` - The developer plan's active-user limit has been reached.

`expired key` - The user's key has expired.

`hwid` - The supplied hardware identifier does not match the key's recorded HWID.

`outdated` - The provided version does not match the system version. No heartbeat token is issued. Send `bypass` as `version` when version checking is not needed.

`digest` - The Program Hash is missing or does not match the value configured for the system.

`beat rate must be over 25 seconds` - The requested beat rate is too fast.

`beat rate must be less than 3600 seconds (1 hour)` - The requested beat rate is too slow.

A successful initial request is formatted as `token|identifier|timestamp:hash`. `token` is the heartbeat token and begins with `TT`; `identifier` is the authenticated username for an account-based request or the supplied key for a key-only request. Confirm that the identifier matches the one you sent to protect against replay attacks. `timestamp` is the current Unix timestamp divided by 29 and rounded down; verify it against the same calculation on the client. `hash` is the SHA-1 hash of everything before the colon. Only a response whose token begins with `TT` is successful.

#### Heartbeat request

Send heartbeat requests to `https://systemlocker.net/quicksilver/beat` at the exact interval accepted during initialization (likely 30 seconds). Each successful heartbeat returns a replacement token; save it and use it for the next heartbeat request.

##### Request body

`token` - The heartbeat token from the successful initial request or the previous successful heartbeat response.

`system` - The system ID used to create the session.

##### Response body

`bad request` - The `token` or `system` field was missing.

`bad session token` - The token is invalid, expired, or has already been used.

`stale token` - The time since the previous heartbeat was too long.

`terminated [reason]` - The developer terminated the session. Remove the `terminated ` prefix to display the developer-provided reason.

`rate limit` - The heartbeat was sent too soon.

`expired key` - The user's key has expired.

A response beginning with `TTr` is the replacement heartbeat token. This is the only successful heartbeat response; use that new token for the next heartbeat request.

### Variable

Server-side variables may be used without authentication. No user information is required; only your system ID and the variable name. This feature can be reached at `https://systemlocker.net/auth/variable`

#### Request body

In order to authenticate, the following pieces of data must be provided:

`system` - This is the system ID. It's a 20 character alphanumeric string. You can find it on any page where your system is listed.

`variable` - The name of the server-side variable you would like to see.

`key` - Optional. A valid license key for the system. Required when accessing a variable that has been marked as protected. If the variable is protected and no valid key is provided, the response will be the same as if the variable was not found.

`clean` - Optional. Set to `1` or `true` and the server will not use JSON for the `false` or success response.

#### Response body

There are a number of possible responses. One of these values will be in the response body:

`no sys` - There was no system provided in the request.

`no var` - There was no variable name provided in the request.

`dbe` - Something failed on the server.

The final two possible responses will be provided in JSON format if `clean` was not set (see above).

If the variable could not be found in the database, your response will be `{"intent":false}`. If `clean` was set, it will be `false`.

If the variable was found in the database, your response will be `{"intent":true,"var_VARIABLENAME":"VALUE"}` where `VARIABLENAME` is the value provided in the `variable` field of your request and `VALUE` is the value of the variable. If `clean` was set, the response will only contain the value of your variable.

### Management

The management API is useful for providing HWID resets and creating keys without visiting the website. It can be reached via HTTP POST at `https://systemlocker.net/api/v1`

The legacy path `https://systemlocker.net/api/endpoint2.php` is also accepted for now.

**This API is potentially dangerous, since it allows keys to be created remotely. Protect your API key at all costs.**

It is common to add these features to a Discord bot: ensure that only the correct people can use your bot. Support requests related to the API may take longer to answer.

#### Request body

Every request needs the following field:

`key` - Your system's API key. This is distinct from your system ID. You can find or regenerate it on your system settings page.

To query data, add the `select` field. Accepted values for `select` are:

- `users` — Returns the count of redeemed keys for the system.
- `key` — Returns the redemption status of the key specified in `lkey`.
- `expiration` — Returns the expiration date of the key specified in `lkey`.

To perform an action, add the `command` field. Accepted values for `command` are:

- `hwidreset` — Resets the HWID for the key specified in `license`. By default this bypasses the 30-day cooldown (admin behavior). To enforce the cooldown, add `as_admin` with the value `false`.
- `genkeys` — Generates one or more keys. Optional parameters: `expire` (0–5, where 0 is permanent and 1–5 correspond to 1 day, 1 week, 1 month, 3 months, and 1 year), `note` (string up to 250 characters), and `count` (number of keys, up to 100).
- `bankey` — Permanently deletes the key specified in `license`.
- `adjustexpiry` — Adjusts the expiry date for the key in `license`. Provide `newexpiry` (date string) and `tz` (timezone string, e.g. `America/Chicago`). Set `newexpiry` to `0` to make a key permanent.
- `systemhwidreset` — Resets the HWID for every key in the system.

#### Response body

There are a number of possible responses. A successful response will have an HTTP 200 status and not have the `error` header set to `true`. Anything else is a failure.

Because all of the responses are human-readable, it is recommended to display them directly to the user.
