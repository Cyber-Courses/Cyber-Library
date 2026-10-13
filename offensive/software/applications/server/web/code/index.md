---
title: "Application code vulnerabilities"
order: 1
description: "Flaws that originate in the application source the development team controls: unsafe query and command composition, broken authorization, and business-logic errors."
keywords:
  - application code
  - business logic
  - authorization
  - injection
  - secure coding
---

# Code

This subtree covers vulnerabilities that originate in the **application code** the development team controls: unsafe composition of queries and commands, broken authorization and access control, logic and workflow flaws, and misuse of frameworks. These would remain even if the web server and runtime were swapped for secure defaults, because the fault is in the source.

Server and reverse-proxy misconfiguration belong to the platform layer, and interpreter or engine bugs belong to the runtime layer; this section is for “same server, same runtime, vulnerable app code.”

## Subtopics

- **[Access control](access-control/index.md)**: Broken access control in server-side web code: endpoint, object, and property scope, plus trust toward headers and upstream systems.
- **[Authentication](authentication/index.md)**: An offensive guide to breaking how server-side code proves identity (credential, federated, MFA, session, and token handling) with the recon and attack metho...
- **[Identification](identification/index.md)**: How application behavior reveals which accounts exist, eases credential stuffing, and exposes user identifiers, the pre-authentication recon that feeds accou...
- **[Injection](injection/index.md)**: Untrusted data bound into something the server executes, parses, stores, renders, or requests, organized by the dangerous primitive under attack.
- **[Workflow](workflow/index.md)**: Business-logic and state integrity in multi-step and multi-service web flows: sequencing, concurrency, and parameter integrity.
