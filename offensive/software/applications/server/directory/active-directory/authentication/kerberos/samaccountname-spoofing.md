---
title: "sAMAccountName spoofing: the noPac escalation"
description: "Escalating from a standard domain user to domain admin by renaming a controlled computer account to match a domain controller, exploiting how the KDC resolves principals during the S4U2self TGS exchange (noPac)."
keywords:
  - sAMAccountName spoofing
  - noPac
  - sAMAccountName
  - S4U2self
  - machine account quota
---

# sAMAccountName spoofing

This escalation (commonly "noPac") abuses how the KDC looks up accounts by name. If a computer account is renamed so its `sAMAccountName` matches a **domain controller's** name (without the trailing `$`), the KDC can be made to issue a service ticket as though the request came from the DC. From there S4U2self yields a ticket impersonating a Domain Admin. It takes a standard user and, where unpatched, ends at domain compromise.

## The sequence

```bash
# One-shot (Impacket-based noPac / sam_the_admin)
noPac.py example.local/user:pass -dc-ip <dc> -impersonate Administrator -shell
```

Under the hood:

1. Create a computer account (default machine account quota allows it), e.g. `EVIL$`.
2. Clear its SPNs (an SPN would block the rename) and **rename** its `sAMAccountName` to the DC's name without `$`, e.g. `DC01`.
3. Request a **TGT** for that name. The TGT is issued for `DC01`.
4. Rename the account back (so `DC01` no longer resolves to it). Now the TGT references a principal the KDC re-resolves to the **real DC**.
5. Use **S4U2self** with that TGT to request a service ticket as `Administrator` to the DC, obtaining a ticket that impersonates a Domain Admin on the DC.

## Why it works

The flaw is that the KDC, handling S4U2self, resolves the client by name and, when the original principal no longer exists, falls back to the matching DC account, binding the ticket to the DC's privileges. The missing PAC validation in the vulnerable versions is what the "noPac" name refers to.

## Exploitation notes

- Needs only a **standard domain user** and a non-zero **machine account quota** (default 10), which makes it a severe privilege-escalation primitive where patching is behind.
- The resulting ticket impersonates a Domain Admin to the DC: use it for [DCSync](../credentials/ntds-and-dcsync.md) or direct execution.
- If the quota is 0, the attack needs an existing computer account you already control to rename.
- Tooling handles the race between renaming and requesting; doing it by hand requires careful timing of steps 2 to 4.

## Tools

- **noPac.py / sam_the_admin.py**: automated end-to-end exploitation.
- **Impacket** (`addcomputer.py`, `renameMachine.py`, `getST.py`): the manual steps.

## References

- [Sophos: noPac, a tale of two vulnerabilities](https://www.sophos.com/en-us/blog/nopac-a-tale-of-two-vulnerabilities-that-could-end-in-ransomware)
- [Impacket (addcomputer / renameMachine / getST)](https://github.com/fortra/impacket)
- [The Hacker Recipes: sAMAccountName spoofing](https://www.thehacker.recipes/ad/movement/kerberos/samaccountname-spoofing)
