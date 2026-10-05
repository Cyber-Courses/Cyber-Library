---
title: "Zimbra: attacking the Collaboration Suite web client, admin console, and SOAP"
description: "Zimbra Collaboration Suite as an attack surface: the mailboxd (Jetty) application behind an nginx proxy, the web client on 443 and admin console on 7071, and the SOAP API at /service/soap and /service/admin/soap. Fingerprint the build, then move through account enumeration, admin-token abuse, and the unauthenticated RCE chains."
keywords:
  - zimbra
  - zimbra collaboration suite
  - zimbra soap
  - mailboxd
  - zimbra admin console
---

# Zimbra

Zimbra Collaboration Suite (ZCS) is a mail and groupware platform whose server side is a Java application, **mailboxd**, running in Jetty, fronted by an **nginx** proxy and supported by **memcached** (route and auth caching), **amavis/clamav** (mail scanning), and MariaDB. Users reach the web client on 443 under `/zimbra/`; administrators use a separate console on **7071** under `/zimbraAdmin/`. Almost every server function, user and admin alike, is driven by a **SOAP** API: user operations at `/service/soap`, privileged operations at `/service/admin/soap`, with authentication carried by the `ZM_AUTH_TOKEN` (user) and `ZM_ADMIN_AUTH_TOKEN` (admin) cookies.

That architecture is why Zimbra is a rich target. The nginx-to-mailboxd proxying, the SOAP token model, and the many extensions (`mboximport`, Autodiscover, the ProxyServlet) have each produced reachable, often unauthenticated, paths to a web shell or an admin token. The SOAP API also leaks account information before authentication, giving a clean enumeration step that feeds everything after it.

## Triage

Confirm Zimbra and pin the build; the RCE chains are strictly version-bound.

```bash
curl -sk https://<target>/zimbra/ -i | grep -iE 'set-cookie|location'   # user client
curl -sk https://<target>:7071/zimbraAdmin/ -i | head                   # admin console
curl -sk https://<target>/service/admin/soap -i | head                  # admin SOAP endpoint
# version from client asset paths
curl -sk https://<target>/zimbra/ | grep -oE 'skins|/js/[0-9]+\.[0-9]+[^"]*' | head
```

The cookies `ZM_AUTH_TOKEN`/`ZM_ADMIN_AUTH_TOKEN`, the `/service/soap` and `/service/admin/soap` endpoints, and the 7071 admin console together confirm ZCS. The build string surfaces in the client JS asset paths and, with local access, from `zmcontrol -v`. Record the exact patch level: it decides which of the RCE chains below is live.

## Pages

- **[Enumeration](enumeration.md)**: account and version discovery through the public SOAP endpoint and login differentials before you hold any token.
- **[Authentication bypass](authentication-bypass.md)**: obtaining or forging a valid auth token, including admin-to-user impersonation via `DelegateAuthRequest` and the token-handling flaws that mint a token without credentials.
- **[RCE chains](rce-chains.md)**: the unauthenticated `mboximport` web-shell write, the memcached CRLF response injection, the ProxyServlet SSRF to the admin SOAP, and the Autodiscover XXE, each ending in code execution on the server.

## References

- [HackTricks: pentesting Zimbra / web services](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
- [Zimbra SOAP API reference](https://wiki.zimbra.com/wiki/SOAP_API_Reference_Material_Beginning_with_ZCS_8.0)
- [Zimbra security advisories](https://wiki.zimbra.com/wiki/Zimbra_Security_Advisories)
