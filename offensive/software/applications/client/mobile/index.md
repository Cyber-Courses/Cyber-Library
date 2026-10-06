---
title: "Mobile: attacking Android and iOS applications"
description: "Offensive techniques against mobile applications: insecure local storage of secrets, exported components and insecure interprocess communication, deep-link and URL-scheme abuse, weak transport and certificate pinning, and bypassing the platform sandbox and root/jailbreak and tampering checks on Android and iOS."
keywords:
  - mobile application security
  - Android
  - iOS
  - insecure storage
  - deep links
---

# Mobile

A mobile application runs inside a per-app sandbox on a device the user carries, and much of its attack surface is what it stores locally, what it exposes to other apps on the device, and how it talks to its backend. The attacker position ranges from another app on the same device to a user who controls (and can root or jailbreak) their own phone to inspect and tamper with the app.

## The surface

- **Insecure storage**: secrets, tokens, and personal data left in shared preferences, databases, files, or the keychain/keystore without adequate protection.
- **Interprocess communication**: exported Android components (activities, services, broadcast receivers, content providers) and iOS URL schemes and app extensions that other apps can reach or abuse.
- **Deep links and URL schemes**: links that drive the app into privileged states or smuggle data when opened from outside.
- **Transport**: weak TLS configuration and certificate-pinning bypass that expose or alter traffic.
- **Platform integrity**: defeating root/jailbreak detection, debugger and tamper checks, and the sandbox that confines the app.

## Seams

The mobile app's server-side API is attacked as a [Server](../../server/index.md) target; this area is the on-device application.

## References

- [OWASP Mobile Application Security (MASVS/MASTG)](https://mas.owasp.org/)
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
