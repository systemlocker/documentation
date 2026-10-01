# System Locker Documentation

Reference documentation for the System Locker authentication, authorization, server-side variable, and management APIs. The same documentation is rendered at [systemlocker.net/documentation](https://systemlocker.net/documentation).

## Documentation index

The documentation is split into focused pages:

- [Simple API](docs/simple-auth.md) for direct checks from trusted server-side environments.
- [Bedrock](docs/bedrock.md) for connected software running on user-controlled hardware.
- [Nightflyer](docs/nightflyer.md) for software that must keep working offline after authorization.
- [Quicksilver](docs/quicksilver.md) for existing production session-authentication integrations.
- [Server-side Variables](docs/variables.md).
- [Management API](docs/management-api.md).
- [SL-HWID](docs/sl-hwid.md) for the fault-tolerant hardware identifier used by Bedrock and Nightflyer.

Every request to System Locker APIs must be an [HTTP POST request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST). Body parameters vary by endpoint.

Account authentication supports Google SSO on the Simple API, Quicksilver, and Bedrock. Libraries must surface the SSO link, obtain the resulting system-specific password from the customer, and retry authentication as described in [Simple API](docs/simple-auth.md#google-sso-account-password) and [Bedrock](docs/bedrock.md).

## Client libraries

Official client libraries that implement these APIs:

| Library | Protocol | Language |
| ------- | -------- | -------- |
| [System Locker Bedrock C++](https://github.com/systemlocker/System-Locker-Bedrock-CPP) | Bedrock | C++ |
| [System Locker Bedrock .NET](https://github.com/systemlocker/System-Locker-Bedrock-.NET) | Bedrock | C# / .NET |
| [System Locker Bedrock Go](https://github.com/systemlocker/System-Locker-Bedrock-Go) | Bedrock | Go |
| [System Locker Bedrock NodeJS](https://github.com/systemlocker/System-Locker-Bedrock-NodeJS) | Bedrock | Node.js |
| [System Locker Bedrock Python](https://github.com/systemlocker/System-Locker-Bedrock-Python) | Bedrock | Python |
| [System Locker Simple C++](https://github.com/systemlocker/System-Locker-Simple-CPP) | Simple API | C++ |
| [System Locker Simple .NET](https://github.com/systemlocker/System-Locker-Simple-.NET) | Simple API | C# / .NET |
| [System Locker Simple Go](https://github.com/systemlocker/System-Locker-Simple-Go) | Simple API | Go |
| [System Locker Simple NodeJS](https://github.com/systemlocker/System-Locker-Simple-NodeJS) | Simple API | Node.js |
| [System Locker Simple Python](https://github.com/systemlocker/System-Locker-Simple-Python) | Simple API | Python |

Pick a **Bedrock** library for software running on machines you don't control: every response is Ed25519-signed and verified against a pinned public key, with rolling session tokens and heartbeats. Bedrock libraries also include Invisible Folder file delivery (`download`, `downloadToFile`, `downloadIfNew`) for protected downloads and auto-updates.

Pick a **Simple** library when one stateless check per action is enough. The Simple libraries also wrap the [Management API](docs/management-api.md) for key generation, expiry adjustment, and HWID resets from your own tooling.

Use **Nightflyer** when desktop software must keep working without a connection after its initial authorization; see the [Nightflyer reference](docs/nightflyer.md). The [SL-HWID library](https://github.com/systemlocker/SL-HWID) provides the fault-tolerant hardware identifier used by Bedrock and Nightflyer, standalone for C++20 and .NET 8 applications.
