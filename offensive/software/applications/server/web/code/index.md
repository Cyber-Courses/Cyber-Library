---
title: "Web application code security: authorization, injection, business logic, and framework misuse in the app layer"
description: Vulnerabilities in server-side web application code—business rules, authorization, unsafe data composition, and framework misuse in the app layer.
keywords:
  - application code vulnerabilities
  - business logic security
  - web authorization
  - server-side web security
---

# Web application code

This section covers issues that **originate in application code** the team maintains: business rules, authorization checks, unsafe composition of queries or commands, logic flaws, and misuse of frameworks and libraries. The mental model: if you could swap the web **server** and **runtime** for secure defaults but keep the same **source repository**, the weakness would still be there.

**Out of scope as the primary home here:** web server or reverse-proxy misconfiguration only; module bugs in `httpd` or `nginx` without an application hand-off problem; pure interpreter or VM bugs that are not about how *your* code called APIs (cross-link when both apply).

Readers should know how HTTP requests reach a route handler, then progress from common **injection** and **access control** mistakes to subtler **workflow** and **state** issues.

## Major branches


| Branch                                    | Focus                                                                                                                    |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| [Access control](access-control/index.md) | Who may invoke which function and which object; endpoint, object, and property levels; trust toward proxies and headers. |
| [Authentication](authentication/index.md) | Proving identity in code: credentials, flows (OAuth, SAML, WebAuthn), sessions, and tokens.                              |
| [Identification](identification/index.md) | Account discovery, brute-force surfaces, and identifier exposure in application behavior.                                |
| [Injection](injection/index.md)           | Where untrusted data is bound into something the process executes, parses, stores, or sends.                             |
| [Workflow](workflow/index.md)             | Sequencing, parallel effects, and integrity of business parameters across multi-step and multi-service flows.            |


## See also

- [Web (parent)](../index.md)