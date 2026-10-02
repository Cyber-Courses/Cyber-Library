---
title: "Port selection and scanning through an SSRF sink"
description: "Varying the port in a fetchable URL turns a blind SSRF into an internal port scanner, using response timing and error differences to map which services are listening."
keywords:
  - SSRF port scan
  - internal port scanning
  - response timing oracle
  - connection refused
  - service discovery
---

# Port

The port in the authority decides which service on the host answers. By iterating it against an internal address, a fetch sink becomes a port scanner that maps the internal network from the server's vantage point, even when no response body is returned.

## The timing and error oracle

A blind sink returns no content, but the *way* it fails still leaks the port state. Three outcomes usually separate cleanly:

- **Open**: the TCP connection succeeds. The service may speak a protocol the client does not understand, so the request hangs until a read timeout, or returns a protocol or parse error after connecting. Either way the connection phase is fast.
- **Closed**: the host sends an immediate RST, so the client reports connection refused almost instantly.
- **Filtered**: a firewall drops the packet, so the client blocks until a full connect timeout, which is distinctly longer than the other two.

Measuring response time per port, and reading any error string the sink reflects, classifies each port without any application-level reply.

```
http://10.0.0.5:22/      # fast connect, then banner or protocol error  -> open
http://10.0.0.5:3306/    # fast connect                                 -> open
http://10.0.0.5:8081/    # immediate refused                            -> closed
http://10.0.0.5:9999/    # long hang to timeout                         -> filtered
```

## Which ports to sweep

Scanning all 65535 ports through a slow sink is rarely worthwhile; target the services that matter:

```
22 80 443 2375 3306 5432 6379 8080 8443 9200 11211 27017
```

These cover SSH, web front ends, the Docker API, common databases, Redis, Elasticsearch, memcached, and MongoDB. Finding one open narrows what to attack next, and the matching scheme payload (for example [Gopher](../scheme/gopher.md) at `6379`) then interacts with it.

## Loopback versus neighbors

Point the scan at `127.0.0.1` to find services bound only to loopback (admin panels, debug servers, databases listening locally), which are invisible from outside the host but reachable through the server itself. Then pivot to RFC1918 neighbors (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) to map adjacent hosts. Combine with the [IP address](ip-address.md) encodings when the literal address is filtered.

## Tools

- **SSRFmap**: automated internal port scanning and service discovery through an SSRF sink.
- **curl**: iterating a port list against an internal host and reading timing or error differences by hand.
- **nmap**: building a common-service port list to feed into the sink (for example `22,80,443,3306,6379,8080`).

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
