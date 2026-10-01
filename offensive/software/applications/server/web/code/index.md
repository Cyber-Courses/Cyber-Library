---
title: "Application code vulnerabilities"
description: "Flaws that originate in the application source the development team controls — unsafe query and command composition, broken authorization, and business-logic errors."
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
