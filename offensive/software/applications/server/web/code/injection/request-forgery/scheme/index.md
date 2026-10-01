---
title: "Scheme: protocol handlers beyond http in SSRF"
description: "A URL library honors whatever schemes it registers. Beyond http and https, handlers for file, gopher, dict, ldap, ftp, tftp, jar, and netdoc each reach a different class of target."
keywords:
  - SSRF URL scheme
  - gopher
  - file scheme
  - dict
  - ldap
  - protocol smuggling
---

# Scheme

The scheme is the protocol prefix that selects which handler resolves a URL. SSRF defenses concentrate on `http` and `https`, but a URL library services whatever handlers its runtime registers, and each one reaches a different target. Supplying an unexpected scheme is often the difference between a blind HTTP fetch and a direct file read or a raw-byte write to an internal service.

The pages group by what the handler buys an attacker:

- **Local data**: [File](file.md) reads the filesystem through `file://`.
- **Raw TCP to internal services**: [Gopher](gopher.md) writes arbitrary bytes to a TCP port, which smuggles crafted protocol payloads (Redis, SMTP, HTTP) and is the most powerful scheme when available.
- **Service-specific reach**: [DICT](dict.md), [LDAP](ldap.md), [SFTP](sftp.md), and [TFTP](tftp.md) speak to their named services and double as interaction and port probes.
- **HTTP itself**: [HTTP and HTTPS](http-and-https.md) is the baseline scheme, reaching internal web services and the metadata endpoint, and pairs with [Port](../authority/port.md) for scanning.
- **Runtime-specific handlers**: [JAR](jar.md) and [Netdoc](netdoc.md) are Java URL handlers that extend reach on JVM stacks.

## Probing which schemes resolve

Before building a scheme-specific payload, confirm the handler exists. Point each scheme at an attacker-controlled listener or an obviously invalid target and watch the error or interaction: a connection attempt, a protocol-specific error, or a timeout each distinguish a registered handler from one the library rejects outright. The handler set depends on the language, the URL library, and its configuration, so enumeration precedes exploitation.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
