---
title: "Request forgery: server-side URL abuse (SSRF)"
order: 6
description: "When untrusted input steers an outbound request, an attacker reaches internal services, cloud metadata, and alternative URL schemes. This hub decomposes the attack by URL facet and by the client that performs the fetch."
keywords:
  - SSRF
  - server-side request forgery
  - URL sink
  - cloud metadata
  - alternative URL schemes
  - internal service access
---

# Request forgery

Request forgery happens when an application takes a URL, host, or path from untrusted input and issues an outbound request to it. The server becomes a proxy that speaks from inside the network perimeter, so a request the attacker could never send directly now reaches loopback services, link-local metadata endpoints, cloud control planes, and neighbors on the internal network. Server-side request forgery (SSRF) is the common name, but the same primitive drives file reads through `file://`, raw TCP interaction through `gopher://`, and protocol smuggling through the handlers a URL library happens to register.

The feature that exposes it is almost always benign: a webhook sender, a link preview or unfurl, an avatar or image proxy, a PDF or screenshot renderer, an import-from-URL field, a health check, or a document converter. Each of these takes a URL and fetches it, and each is a sink.

## How this section is organized

A URL has distinct parts, and a validator usually trusts some while attacking is possible through others. The subtrees follow those parts:

- **[Authority](authority/index.md)**: the host and port. Loopback and link-local targets, alternate IP encodings, hostname tricks (DNS rebinding, internationalized domains), and port reach.
- **[Path](path.md)**: traversal and normalization inside the URL path, used to escape a fixed base URL or confuse an allowlist that only checks a prefix.
- **[Query](query/index.md)**: parameter-level bypasses, including second-order redirects and HTTP parameter pollution that defeat an allowlist after it has already passed.
- **[Scheme](scheme/index.md)**: the protocol handler. Beyond `http(s)`, a URL library may honor `file`, `gopher`, `dict`, `ldap`, `ftp`/`sftp`, `tftp`, `jar`, and `netdoc`, each reaching a different class of target.

Orthogonal to the URL grammar is **[Fetch](fetch/index.md)**, the question of *how* the stack performs the request: a programmatic HTTP client, or a full headless browser that carries cookies and runs script. The same malicious URL behaves differently in each, so the client is its own axis.

## Why it escalates

A bare fetch that returns nothing visible is still useful: response timing and error differences turn it into a port scanner and a service oracle. When the response *is* reflected, the metadata service hands back instance credentials, internal admin panels return their contents, and `file://` returns local files. When the scheme allows raw bytes on the wire, `gopher://` writes a crafted Redis or SMTP command and converts a read primitive into code execution or mail relay. The facets below are the routes to each of those outcomes.

## Tools

- **SSRFmap**: automated SSRF exploitation across loopback, metadata, and internal targets.
- **Burp Suite**: crafting and replaying fetch requests while swapping URL facets and clients.
- **Burp Collaborator**: confirming blind and out-of-band SSRF through DNS and HTTP callbacks.
- **interactsh**: self-hosted OOB interaction server for detecting blind outbound requests.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [OWASP WSTG: Testing for Server-Side Request Forgery](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
