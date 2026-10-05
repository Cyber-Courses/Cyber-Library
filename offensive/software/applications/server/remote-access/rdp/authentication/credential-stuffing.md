---
title: "Credential stuffing: replaying breached credentials against RDP"
description: "Because RDP accepts Windows credentials and is widely internet-exposed, attackers replay username/password pairs from breaches and prior compromises against it. Unlike brute force, stuffing tries known-valid pairs, so it is low-volume and effective, and it is a dominant RDP initial-access technique precisely because users reuse passwords across services."
keywords:
  - credential stuffing
  - rdp
  - breached credentials
  - password reuse
  - initial access
---

# Credential stuffing

Credential stuffing replays username/password pairs obtained elsewhere, from public breaches, prior intrusions, infostealer logs, against a service in the hope that users reused them. RDP is a prime target: it is massively internet-exposed, it accepts ordinary Windows/domain credentials, and password reuse between personal and corporate accounts is rampant. Stuffing differs from brute force in that it tries a small set of known-valid pairs rather than guessing, so it stays under lockout thresholds and is quiet, which is exactly why it is one of the dominant RDP initial-access techniques behind ransomware intrusions.

```bash
# replay breach/stealer username:password pairs (1:1, not cross-product)
nxc rdp <target> -u leaked_users.txt -p leaked_pass.txt --no-bruteforce   # paired
hydra -C leaked_combos.txt rdp://<target> -t 1       # combo file user:pass
# source pairs from infostealer logs, breach compilations, and prior-compromise creds
```

## Exploitation notes

- Stuffing uses known pairs, so volume is low and lockout risk is minimal; feed it from infostealer logs (which often include the exact RDP host and credentials), breach datasets, and credentials harvested earlier in the engagement.
- Match the username format to the target (local `host\user`, domain `DOMAIN\user`, or UPN) using the domain learned from [enumeration](../enumeration/banner-grabbing.md).
- Infostealer logs are especially potent for RDP because they frequently capture the saved RDP credential and the target address together, making the "guess" a direct replay.
- A hit is interactive access with that user's privileges; as with brute force, expect reuse across the environment and test accordingly.

## References

- [CISA: RDP and credential-based intrusions](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [HackTricks: RDP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
