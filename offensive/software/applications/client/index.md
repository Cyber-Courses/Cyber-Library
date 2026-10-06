---
title: "Client: attacking code that runs on the user's device"
description: "Offensive techniques against client-side software, the code that runs on the user's own device: web front ends in the browser, mobile apps on Android and iOS, and desktop applications. The attacker often controls the execution environment, so protections are advisory and the goal is to turn the client against its user or use it to reach the server."
keywords:
  - client-side security
  - browser security
  - mobile application security
  - desktop application security
  - XSS
---

# Client

Client software runs on the user's own device, which inverts the usual trust assumption: the attacker frequently controls, or can inspect and tamper with, the environment the code runs in. Protections on the client are therefore advisory, and the offensive questions are what the client trusts, what secrets it holds locally, and how it can be turned against its own user or used as a stepping stone to the server it talks to.

This area is organized **by client type**, because the execution model and the defenses differ sharply: a browser enforces the same-origin policy and a sandbox around untrusted web code, a mobile OS confines each app and brokers its interprocess communication, and a desktop application runs with the user's own privileges and local reach.

## Client types

- **[Browser](browser/index.md)**: web front ends executing in the browser, where cross-site scripting, request forgery, and the same-origin, CORS, and content-security-policy model decide what attacker-supplied script can do.
- **[Mobile](mobile/index.md)**: Android and iOS applications, attacked through insecure local storage, exported components and interprocess communication, deep links, and the platform sandbox and transport protections.
- **[Desktop](desktop/index.md)**: installed desktop applications, attacked through local IPC and protocol handlers, insecure update channels, embedded-browser and Electron weaknesses, and secrets stored on the endpoint.

## Seams

The server side of a web application, where input handling, authentication, and the platform, runtime, and code layers live, is under [Server > Web](../server/web/index.md). This area covers only the client half: what runs in the browser, on the phone, or on the desktop.

## References

- [OWASP Web Security Testing Guide: client-side testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/)
- [OWASP Mobile Application Security](https://mas.owasp.org/)
