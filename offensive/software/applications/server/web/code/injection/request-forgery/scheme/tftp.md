---
title: "TFTP scheme SSRF for config retrieval"
description: "The tftp:// scheme makes a curl-backed client issue a UDP read request to an internal TFTP server, which often holds network-device configuration and boot files."
keywords:
  - tftp scheme
  - tftp://
  - UDP 69
  - config retrieval
  - network device
  - octet mode
---

# TFTP

`tftp://` drives a Trivial File Transfer Protocol request over UDP port 69 from a curl-backed client. TFTP needs no authentication, so where an internal TFTP server exists, this scheme reads files straight out of it, which on internal networks frequently means switch, router, and PXE boot configuration.

## Reading a file

The URL names the host, optional port, and the filename to fetch:

```
tftp://10.0.0.5/running-config
tftp://127.0.0.1:69/startup-config
tftp://10.0.0.5/pxelinux.cfg/default
```

The client sends a read request for the named file in octet mode and returns the contents. Because TFTP is unauthenticated and request-response, a single `tftp://` fetch retrieves whatever the server will serve, with device configuration files (which often embed credentials and SNMP strings) the prime target.

## Reach and limits

TFTP has no listing and no authentication, so success depends on naming a file the server exposes; the common config filenames above are the usual guesses. The scheme is UDP, so a non-response is ambiguous (dropped versus absent), making it less reliable as a port oracle than the TCP schemes, but a returned file is unambiguous disclosure. Use it when a [Port](../authority/port.md) probe or network context suggests TFTP is present, typically alongside network infrastructure.

## Tools

- **curl**: manual `tftp://` read requests for common config filenames over UDP port 69.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [curl: supported protocols](https://curl.se/docs/url-syntax.html)
