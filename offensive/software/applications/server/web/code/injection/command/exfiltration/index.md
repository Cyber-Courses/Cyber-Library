---
title: "Blind command injection exfiltration"
description: "Recovering output when none is returned: time-based oracles and out-of-band DNS/HTTP channels."
keywords:
  - blind command injection
  - out-of-band
  - OOB exfiltration
  - time-based
  - DNS exfiltration
---

# Exfiltration

When a command runs but its output never appears in the response, data is recovered through side channels. A **time-based** oracle makes execution conditional on a guess and measures a deliberate delay; an **out-of-band** channel makes the target reach a host you control (DNS or HTTP) and carries command output in the hostname or request path. These are the same inference techniques used in blind SQL injection, applied to shell output.

## Pages

- **[DNS out-of-band channel](dns.md)**: Encoding command output into a DNS subdomain label so it leaves a blind OS command injection target via name resolution, captured by interactsh or a dnsbin s...
- **[Time-based blind extraction](time-based.md)**: Using sleep and ping delays as a boolean oracle in blind OS command injection to confirm execution and extract data character by character when no output and...

## Tools

- **[commix](https://github.com/commixproject/commix)**: automates time-based and out-of-band blind extraction.
- **[interactsh](https://github.com/projectdiscovery/interactsh)**: self-hostable DNS and HTTP callback capture.
- **Burp Collaborator**: integrated out-of-band interaction catcher.

## References

- [PortSwigger Web Security Academy: Blind OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
