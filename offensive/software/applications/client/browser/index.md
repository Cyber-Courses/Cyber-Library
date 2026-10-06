---
title: "Browser: attacking web front ends in the browser"
description: "Offensive techniques against web code running in the browser: cross-site scripting and its execution contexts, cross-site request forgery, clickjacking and UI redress, and abusing or bypassing the same-origin policy, CORS, PostMessage, WebSockets, and the Content Security Policy that are supposed to contain untrusted web code."
keywords:
  - cross-site scripting
  - CSRF
  - same-origin policy
  - CORS
  - content security policy
---

# Browser

The browser runs untrusted code from every site at once and is supposed to keep each origin isolated from the others and from the user's data. Client-side web attacks live in the gaps of that model: getting attacker script to run in a trusted origin, making the victim's browser act against its will, and abusing the cross-origin messaging and resource-sharing mechanisms that relax isolation on purpose.

## The surface

- **Cross-site scripting**: running attacker script in a trusted origin (reflected, stored, and DOM-based), its execution contexts and sinks, and the filter and sanitizer bypasses that reach them.
- **Request forgery and UI redress**: cross-site request forgery, clickjacking, and other ways of making the victim's authenticated browser perform actions.
- **The origin model**: the same-origin policy and its boundaries, CORS misconfiguration, `PostMessage` and cross-document messaging, WebSockets, and cross-site leaks.
- **Containment**: bypassing or weakening the Content Security Policy and other client-side defenses, and abusing browser storage and extensions.

## Seams

Server-side injection, authentication, and logic flaws, including where a web application's server reflects or stores the data that becomes an XSS payload, are under [Server > Web](../../server/web/index.md); this area is the browser-side execution and the origin model.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP WSTG: client-side testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/)
