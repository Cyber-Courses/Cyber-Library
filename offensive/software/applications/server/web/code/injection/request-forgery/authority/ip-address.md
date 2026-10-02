---
title: "SSRF IP address literals and encodings: loopback, decimal and hex forms, IPv6, and cloud metadata endpoints"
description: IP literals, encodings, and special ranges in outbound URL fetches for SSRF testing in application code.
keywords:
  - SSRF
  - 127.0.0.1
  - IPv6
  - decimal IP
---

# IP literals (SSRF)

## Context

Application URL parsers and HTTP clients accept many spellings of the same host: dotted decimal, integer form, hex, octal, IPv6 with IPv4-mapped forms, and compressed IPv6. SSRF testing maps which spellings the library resolves to loopback or link-local when the attacker supplies the URL.

## Theory

`127.0.0.1`, `2130706433`, `0x7f000001`, `0177.0.0.1`, `::ffff:127.0.0.1`, and `localhost` may normalize differently between DNS, Java `InetAddress`, Python `urlparse`, and curl. Metadata IPs (`169.254.169.254` on clouds) are targets when egress allows. The offensive task is to find a spelling the app accepts and the policy forgot to block.

## Practice

### Rotate literals on one in-scope URL field

- In a lab or written-authorized test, POST the same callback URL with host `127.0.0.1`, then `2130706433`, then `[::ffff:127.0.0.1]`, and observe server-side logs or blind callbacks to your collector if the app exfiltrates response bodies.

### Compare parser vs curl

- If the app uses Node `url.parse` + `http.get`, verify behavior differs from `curl` on the same string; document the exact client class for the report.

## Tools

- **Burp Suite Collaborator**
- **curl**
- **Python** `http.client`