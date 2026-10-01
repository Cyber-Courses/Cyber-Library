---
title: "Applications: offensive techniques by where the code runs"
description: "The applications category splits by execution location, client-side and server-side, because the trust model and the reachable attack surface differ fundamentally between code the user controls and code the operator controls."
keywords:
  - application security
  - client-side attacks
  - server-side attacks
  - web application security
  - trust boundary
---

# Applications

The applications category covers offensive work against the programs that implement an organization's behavior, as opposed to the operating systems that host them (its sibling under [Software](../index.md)). It is the densest part of the library, because applications expose the most surface to untrusted input and change the most often.

## Why it is split this way

The two subcategories divide by where the application code executes, which decides who controls it and what an attacker can reach:

- **Client**: code that runs on the user's own device (browsers, desktop, and mobile apps). The attacker frequently controls the execution environment, so the questions are about what the client trusts, what secrets it holds, and how it can be turned against its own user or used to reach the server.
- **[Server](server/index.md)**: code that runs on infrastructure the operator controls. The attacker reaches it only through the interfaces it exposes, so the questions are about input handling, authentication and authorization, and the services the server stitches together.

The split reflects a fundamental difference in trust. On the client, the attacker often owns the machine, so protections are advisory and the goal is often to influence the server or another user. On the server, the attacker is remote and constrained to the exposed interface, so the goal is to make that interface do something it should not. A single assessment usually crosses the boundary, since client behavior shapes what reaches the server, but the two sides demand different assumptions, which is why the category branches here before going deeper.

## References

- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
