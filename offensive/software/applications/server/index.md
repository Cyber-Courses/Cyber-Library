---
title: "Server: offensive techniques by server service type"
description: "The server category organizes attacks by the kind of service under test, from web and database to directory, messaging, and cloud, because each service class has its own protocols, primitives, and abuse patterns."
keywords:
  - server security
  - web application security
  - database attacks
  - directory services
  - cloud security
---

# Server

The server category covers offensive work against code and services that run on infrastructure the operator controls, reached remotely through the interfaces they expose. It is organized by the kind of service, because a web front end, a database, a directory, and a message broker each speak different protocols and fail in different ways.

## Why it is split this way

The subcategories group by service class, so a technique sits next to the protocol and data model it actually abuses:

- **[Web](web/index.md)**: HTTP-facing applications and the platform, runtime, and code layers that serve them. The broadest subtree, and where most application-level injection and logic flaws live.
- **Database**: SQL and NoSQL engines, their query languages, authentication, and exposed management surfaces.
- **Directory**: identity and directory services (LDAP, Active Directory), where enumeration and privilege abuse dominate.
- **Messaging**, **File Share**, **Remote Access**, **Monitoring**, **Versioning**, **Security**: the supporting services an environment runs, each attacked through its own protocol and trust assumptions.
- **Cloud**, **Containers**, **Virtualization**: the hosting and isolation layers, where misconfiguration and escapes cross tenant and workload boundaries.
- **Network**: server-side network services as a target in their own right, distinct from the medium-level work in the top-level [network](../../../network/index.md) category.

Splitting by service type matches how an operator triages a host: enumerate what is listening, identify each service, then reach for the techniques specific to it. Keeping the classes separate stops a database technique from being filed next to a message-queue one, since the query languages, authentication models, and abuse primitives have nothing in common. The web subtree is split further by layer (platform, runtime, and code) because a single web service combines all three.

## References

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK](https://attack.mitre.org/)
