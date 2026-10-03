---
title: "Golden gMSA: forging managed service account passwords offline"
description: "Dumping the KDS root key to compute the password of any group Managed Service Account offline and forever, the Golden gMSA attack, a gMSA equivalent of the golden ticket with no way to rotate the underlying secret."
keywords:
  - golden gMSA
  - KDS root key
  - GoldenGMSA
  - msDS-ManagedPasswordID
  - domain persistence
---

# Golden gMSA

A [gMSA](gmsa.md) password is not stored; it is **computed** from the forest's **KDS root key** plus attributes of the account (`msDS-ManagedPasswordID`). Normally the DC does that computation and only authorised hosts retrieve the result. The **Golden gMSA** attack takes the KDS root key itself: dump it once, and you can compute the current and all future passwords of **every** gMSA tied to that key, **offline**, without ever contacting a DC. It is the gMSA analogue of the [golden ticket](../authentication/kerberos/forged-tickets.md), and it is worse in one way: unlike `krbtgt`, the **KDS root key cannot be rotated**, so a compromised key is a permanent exposure.

## The attack

Dumping the KDS root key needs high privilege (Domain Admin, or SYSTEM on a DC) once; after that, computation is unprivileged and offline:

```bash
# Semperis GoldenGMSA: read the KDS root key (privileged, once)
GoldenGMSA.exe kdsinfo
# Enumerate gMSAs and their managed-password IDs
GoldenGMSA.exe gmsainfo
# Compute a gMSA's password fully offline: SID + its managed-password ID (from gmsainfo) + the root key
GoldenGMSA.exe compute --sid <gMSA-SID> --pwdid <managed-password-id> --kdskey <base64-kds-key>
```

The output is the gMSA's password (and thus its NT hash / Kerberos keys), used with [pass-the-hash](../authentication/ntlm/pass-the-hash.md) or [overpass-the-hash](../authentication/kerberos/pass-the-key-and-overpass-the-hash.md).

## Why it is durable

- The **KDS root key never rotates**: there is no equivalent of a double `krbtgt` reset, so once the key leaks, every associated gMSA stays forgeable until each gMSA is migrated to a new key.
- Computation is **offline**, so it produces **no DC authentication logs**, unlike legitimately retrieving `msDS-ManagedPassword`.
- It covers **all** gMSAs under the key at once, including privileged service accounts, making one key dump a forest-wide service-account skeleton key.

## Exploitation notes

- This is persistence and lateral reach, not initial access: you need the one-time privileged read of the KDS root key (from the DC, or from a backup/`ntds.dit`).
- The delegated-MSA equivalent, **Golden dMSA**, applies the same idea to Server 2025 dMSAs; pair this with the [BadSuccessor](badsuccessor.md) dMSA path when dMSAs are present.
- Target gMSAs that run privileged services; a forged gMSA password is as useful as the service account's rights.

## Tools

- **GoldenGMSA** (Semperis): dump the KDS root key and compute gMSA passwords offline.
- **GoldenDMSA** (Semperis): the delegated-MSA variant for Server 2025.

## References

- [Semperis: introducing the Golden gMSA attack](https://www.semperis.com/blog/golden-gmsa-attack/)
- [GoldenGMSA (Semperis)](https://github.com/Semperis/GoldenGMSA)
- [GoldenDMSA (Semperis)](https://github.com/Semperis/GoldenDMSA)
