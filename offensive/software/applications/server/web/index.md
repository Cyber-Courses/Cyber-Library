---
title: "Web"
description: Offensive application security for HTTP-based services—web apps and APIs—with emphasis on flaws in how developers implement handling, access control, and data processing.
keywords:
  - web application security
  - web pentest
  - HTTP application
---

# Web

**Web** covers software that speaks HTTP (and closely related patterns such as WebSockets on the same application stack). The focus is on behavior implemented in **application source**—routes, handlers, templates, and data access you can reason about from code and traffic—not on running a generic scanner and calling it done.

## Application code

- **[Code](code/index.md)** — Vulnerabilities that originate in **source the development team controls**: authorization mistakes, business-logic abuse, unsafe composition of queries, template misuse, and framework misuse.

## Suggested study order

1. **Identity and access** — Who the server thinks the caller is, and what that identity may do.
2. **Input handling** — How data crosses trust boundaries (injection families by sink).
3. **Workflow and state** — Sequencing, race conditions, and handoffs that span more than one request or service.

## See also

- [Server (parent)](index.md)