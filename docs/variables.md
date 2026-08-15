> [← Documentation home](../README.md) · [Simple Auth](simple-auth.md) · [Bedrock](bedrock.md) · [Quicksilver](quicksilver.md) · **Server-side Variables** · [Management API](management-api.md)

## Server-side Variables

Server-side variables let your application retrieve values managed in the System Locker portal. Send `POST https://systemlocker.net/auth/variable`.

### Request body

`system` - Your 20-character system ID.

`variable` - The variable name to retrieve.

`key` - Optional license key. It is required for variables marked as protected. A protected variable without a valid key is returned as unavailable.

`clean` - Optional. Set to `1` or `true` to receive plain text instead of the JSON response shape.

### Response body

`no sys` - The system is missing.

`no var` - The variable name is missing.

`dbe` - The service could not complete the request.

Without `clean`, an unavailable variable returns `{"intent":false}`. A found variable returns `{"intent":true,"var_VARIABLENAME":"VALUE"}`. With `clean`, an unavailable variable returns `false`, while a found variable returns only its value.
