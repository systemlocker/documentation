# System Locker Documentation

Reference documentation for the System Locker authentication, authorization, server-side variable, and management APIs. The same documentation is rendered at [systemlocker.net/documentation](https://systemlocker.net/documentation).

## Documentation index

The documentation is split into focused pages:

- [Simple Auth](docs/simple-auth.md) for Goliath and Mikros.
- [Bedrock](docs/bedrock.md) for new production session-authentication integrations.
- [Quicksilver](docs/quicksilver.md) for existing production session-authentication integrations.
- [Server-side Variables](docs/variables.md).
- [Management API](docs/management-api.md).

Every request to System Locker APIs must be an [HTTP POST request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST). Body parameters vary by endpoint.

## Client libraries

Official client libraries that implement these APIs:

| Library | Protocol | Language |
| ------- | -------- | -------- |
| [System Locker Bedrock C++](https://github.com/systemlocker/System-Locker-Bedrock-CPP) | Bedrock | C++ |
| [System Locker Bedrock .NET](https://github.com/systemlocker/System-Locker-Bedrock-.NET) | Bedrock | C# / .NET |
| [System Locker Bedrock Go](https://github.com/systemlocker/System-Locker-Bedrock-Go) | Bedrock | Go |
| [System Locker Bedrock NodeJS](https://github.com/systemlocker/System-Locker-Bedrock-NodeJS) | Bedrock | Node.js |
| [System Locker Bedrock Python](https://github.com/systemlocker/System-Locker-Bedrock-Python) | Bedrock | Python |
| [System Locker Simple Go](https://github.com/systemlocker/System-Locker-Simple-Go) | Simple Auth | Go |
| [System Locker Simple NodeJS](https://github.com/systemlocker/System-Locker-Simple-NodeJS) | Simple Auth | Node.js |
| [System Locker Simple Python](https://github.com/systemlocker/System-Locker-Simple-Python) | Simple Auth | Python |

Pick a **Bedrock** library for software running on machines you don't control: every response is Ed25519-signed and verified against a pinned public key, with rolling session tokens and heartbeats. Bedrock libraries also include Invisible Folder file delivery (`download`, `downloadToFile`, `downloadIfNew`) for protected downloads and auto-updates.

Pick a **Simple** library when one stateless check per action is enough. The Simple libraries also wrap the [Management API](docs/management-api.md) for key generation, expiry adjustment, and HWID resets from your own tooling.
