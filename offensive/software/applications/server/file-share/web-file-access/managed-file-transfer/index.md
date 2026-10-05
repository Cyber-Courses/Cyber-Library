---
title: "Managed file transfer: attacking MFT appliances"
description: "Managed file transfer products provide governed business-to-business file exchange through internet-facing web portals and APIs. Their exposure and the sensitive data they broker have made them prime targets: authentication bypasses and injection-to-RCE flaws in MFT appliances have driven some of the largest mass-exploitation and data-theft campaigns, notably against MOVEit, GoAnywhere, CrushFTP, and Serv-U."
keywords:
  - managed file transfer
  - mft
  - moveit
  - goanywhere
  - crushftp
---

# Managed file transfer

Managed file transfer (MFT) products exist to move files between organizations under governance: audit, encryption, retention, and access control, exposed through a web portal and APIs that are, by design, internet-facing. That combination, external exposure plus a concentration of sensitive data from many partners, has made MFT appliances one of the most consequential target classes in recent years. The vulnerabilities cluster in two kinds: authentication bypasses that reach administrative or file functions without credentials, and injection flaws (SQL injection, deserialization, command injection) that escalate to remote code execution. Several have been exploited en masse for data theft.

```bash
# fingerprint the MFT product and version
curl -skI https://<target>/                   # Server/app headers
curl -sk https://<target>/ | grep -iE 'moveit|goanywhere|crushftp|serv-u|filezilla'
```

## Subtopics

- **[Authentication bypass](authentication-bypass.md)**: reaching MFT functions without credentials.
- **[Injection to RCE](injection-to-rce.md)**: SQL injection, deserialization, and command injection.
- **[Known MFT exploits](known-mft-exploits/index.md)**: the specific appliance campaigns.

## References

- [CISA alerts on MFT exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [OWASP: injection and authentication](https://owasp.org/www-project-top-ten/)
