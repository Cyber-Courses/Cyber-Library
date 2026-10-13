---
title: "Enumeration: fingerprinting Nagios and Icinga"
order: 5
description: "Identify whether a target runs Nagios Core, Nagios XI, or Icinga, and its version, from the web interface paths, page content, and the NRPE agent's version response. The product and version decide the attack: Nagios XI's web-application exploits versus Core's file-based configuration versus Icinga Web 2, and the exact build maps to known vulnerabilities."
keywords:
  - nagios enumeration
  - nagios xi
  - icinga web
  - nrpe version
  - fingerprint
---

# Enumeration

The first step is identifying which product and version are present, because the three diverge sharply. Nagios Core exposes a CGI interface (commonly under `/nagios/`) with a recognizable UI and version string; Nagios XI is a full PHP application (under `/nagiosxi/`) with its own login and many more endpoints; Icinga Web 2 (under `/icingaweb2/`) is the Icinga interface. The web paths, page titles, static resources, and footer version strings fingerprint the product and build. The NRPE agent on 5666 also returns its version to a `check_nrpe` query. The product and version select the whole approach: Nagios XI's rich web-application attack surface and known chains, versus Core's command-definition injection, versus Icinga Web 2's own issues.

```bash
# web product/version
curl -sk https://<target>/nagios/ | grep -ioE 'Nagios Core [0-9.]+'
curl -sk https://<target>/nagiosxi/ | grep -i 'nagios xi'
curl -sk https://<target>/icingaweb2/ | grep -i icinga
# NRPE agent version
check_nrpe -H <target>                            # returns "NRPE vX.Y.Z"
```

## Exploitation notes

- The product split is decisive: Nagios XI is a web application with a long vulnerability history ([Nagios XI exploits](nagios-xi-exploits.md)), Nagios Core is configured by files (so attacks center on [command injection](command-and-plugin-injection.md) and [NRPE](nrpe-abuse.md)), and Icinga Web 2 has its own surface.
- The version string maps to known advisories, especially for Nagios XI where version-specific RCE chains are common; fingerprint precisely.
- The NRPE version (from `check_nrpe` with no command) confirms the agent and its generation, which affects whether argument passing (and thus injection) is possible, see [NRPE abuse](nrpe-abuse.md).
- Route to [authentication](authentication.md) for the web interface and to the agent/command surfaces for execution.

## References

- [Nagios XI](https://www.nagios.com/products/nagios-xi/)
- [HackTricks: Nagios](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
