---
title: "Managed file transfer: attacking enterprise MFT appliances"
description: "Attacking managed file transfer (MFT) appliances that organizations use for secure business file exchange: authentication bypass to reach the admin and transfer interfaces, injection flaws leading to remote code execution, and the named exploit chains against MOVEit, GoAnywhere, Serv-U, and CrushFTP that drove mass data-theft campaigns."
keywords:
  - managed file transfer
  - MOVEit
  - GoAnywhere
  - Serv-U
  - CrushFTP
---

# Managed file transfer

Managed file transfer (MFT) appliances provide governed, audited file exchange between organizations, which means they are internet-facing and hold large volumes of sensitive data. That combination has made them a prime target: a single pre-authentication flaw in an MFT product yields the data of every organization running it. Attacks center on authentication bypass, injection to code execution, and the named product exploit chains.

## Subtopics

- **[Authentication bypass](authentication-bypass.md)**: reaching admin and transfer interfaces.
- **[Injection to RCE](injection-to-rce.md)**: injection flaws leading to code execution.
- **[Known MFT exploits](known-mft-exploits/index.md)**: the named product chains.

## References

- [CISA: known exploited vulnerabilities catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [HackTricks: pentesting web](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
