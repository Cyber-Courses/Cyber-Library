---
title: "Cisco ASA and AnyConnect: WebVPN traversal and portal credential attacks"
order: 5
description: "Cisco ASA and Firepower remote-access VPN have had a WebVPN path-traversal giving unauthenticated file disclosure, and the AnyConnect SSL-VPN portal is a persistent target for credential brute force and password spraying. Combined, file disclosure and valid credentials grant access to the VPN and the internal network it fronts."
keywords:
  - cisco asa
  - anyconnect
  - webvpn
  - path traversal
  - password spray
---

# Cisco ASA and AnyConnect

Cisco ASA (and Firepower) provide remote-access VPN through the AnyConnect SSL-VPN, and two avenues recur. The WebVPN interface had a path-traversal vulnerability allowing an unauthenticated attacker to read files from the device, disclosing configuration and other data useful for further access. And the AnyConnect SSL-VPN portal is a continual credential-attack target: because it authenticates against the corporate directory and is internet-facing, password spraying and credential stuffing against it are a common and effective initial-access route, especially where MFA is absent or inconsistently applied. ASA remote-access has also seen targeted campaigns combining these with weak configuration.

```bash
# fingerprint ASA/AnyConnect WebVPN
curl -skI https://<gateway>/
curl -sk https://<gateway>/+CSCOE+/logon.html | grep -i 'anyconnect\|cisco'
# WebVPN path-traversal class: unauthenticated file read of appliance files
curl -sk --path-as-is 'https://<gateway>/+CSCOU+/../+CSCOE+/<traversal-target>'
# portal credential attack: spray/stuff the AnyConnect logon (directory-backed)
```

## Exploitation notes

- The WebVPN traversal is an unauthenticated file read; target device configuration and any disclosed secrets to enable or supplement authenticated access.
- The AnyConnect portal is a prime spraying/stuffing target because it validates corporate credentials and is internet-facing; a sprayed or breached password without MFA is a tunnel inside, see [password brute force](../authentication/password-brute-force.md).
- Watch for auth paths that skip MFA (certain group-URLs, legacy configs); ASA remote-access misconfigurations have enabled MFA-less access even where MFA is nominally deployed.
- Both yield access to the internal network the ASA fronts; fingerprint the ASA/Firepower version for the traversal and identify the AnyConnect group-URLs for credential attacks.

## References

- [Cisco Security Advisories](https://sec.cloudapps.cisco.com/security/center/publicationListing.x)
- [CISA: Cisco ASA remote-access exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
