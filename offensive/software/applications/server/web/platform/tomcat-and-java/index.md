---
title: "Tomcat and Java app servers: AJP, management apps, and path handling"
description: "Platform misconfigurations in Tomcat and Java application servers: the AJP connector (Ghostcat) file read and inclusion, Manager/Host-Manager deployment to RCE, and path-parameter traversal and security-constraint bypass."
keywords:
  - tomcat misconfiguration
  - ghostcat
  - AJP
  - tomcat manager
  - path parameters
---

# Tomcat and Java

Java application servers (Apache Tomcat, and relatives like JBoss/WildFly, Jetty, and GlassFish) carry a misconfiguration set unlike the Apache/nginx/IIS file-servers: a secondary AJP connector, bundled management applications that deploy code, and servlet path handling with its own quirks. Fingerprint Java from the `JSESSIONID` cookie, `X-Powered-By: Servlet`, default Tomcat error pages, and the `/manager`/`/examples` apps, then work this checklist.

## What to check

- **AJP connector (Ghostcat)**: the AJP port (classically `8009`) exposes a protocol that can read or include files under the web app, reaching source disclosure and, with an uploadable file, RCE.
- **Manager and Host-Manager apps**: default or weak credentials on `/manager/html` (and `/host-manager`) allow deploying a WAR, which is direct RCE.
- **Path parameters and normalization**: the `;` path-parameter separator and servlet normalization let crafted paths bypass security constraints and reach protected servlets.

## Pages

- **[AJP and Ghostcat](ajp-ghostcat.md)**: abusing the AJP connector for file read, inclusion, and RCE.
- **[Manager and Host-Manager](manager-and-host-manager.md)**: deploying a WAR for code execution via the management apps.
- **[Path parameters and normalization](path-parameter-and-normalization.md)**: `..;/` and servlet path handling that bypasses access rules.

## References

- Apache Tomcat documentation: AJP connector, Manager app, security
- OWASP WSTG: Testing for application platform configuration
