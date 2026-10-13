---
title: "SCCM credential harvesting: NAA, PXE, and policy secrets"
order: 2
description: "Pulling credentials that Configuration Manager hands to clients: the Network Access Account from machine policy, secrets from unauthenticated PXE boot media, and task-sequence variables, turning any client or any computer account into domain credentials."
keywords:
  - Network Access Account
  - NAA
  - PXE
  - SCCM policy
  - SharpSCCM
---

# Credential harvesting

Configuration Manager has to give clients credentials to do their jobs, and it has historically stored them where a client, or anyone who can register as one, can retrieve and decrypt them. This was the first SCCM attack class to be weaponised and it still works wherever the legacy settings remain.

## The Network Access Account

The **Network Access Account (NAA)** is a domain account SCCM gives clients so they can reach content when their own machine account cannot. It is delivered in **machine policy** and encrypted with a key the client itself holds, so any machine (or any registered computer account) can decrypt it back to a cleartext domain credential:

```bash
# SharpSCCM: request machine policy and recover NAA credentials as SYSTEM on a client
SharpSCCM.exe get naa

# sccmhunter: register a computer, pull policy from the management point, decrypt the NAA
sccmhunter.py http -u user -p password -d example.local -dc-ip <dc> -auto
```

This was the original SCCM "easy win". Microsoft's **Enhanced HTTP** and the move away from the NAA reduce it, so its absence pushes you toward [site takeover](site-takeover.md); its presence is often an instant domain credential.

## PXE boot media

A distribution point configured for **PXE** (operating-system deployment) will hand out boot media whose variables package can carry the NAA and task-sequence credentials. If PXE is not password-protected, or the password is weak, this is **unauthenticated**:

```bash
# PXEThief: pull and decrypt secrets from PXE boot media
python3 pxethief.py 2 <distribution-point-ip>
```

## Task-sequence and collection variables

Task sequences and collection variables frequently embed service-account or local-admin credentials for imaging and installs; once you can read policy or hold Full Administrator, these are a second credential trove.

## Exploitation notes

- The NAA and PXE paths turn **a single client, or just a computer account, into a domain credential**, with no privileged access first.
- Even a **disabled** NAA often remains retrievable from old policy blobs, so pull it regardless of current config.
- PXE without a strong password is a true unauthenticated foothold from the network, so always test it during [recon](reconnaissance.md).

## Tools

- **SharpSCCM** (`get naa`): policy credential recovery.
- **sccmhunter** (`http` module): register a device and pull policy secrets.
- **PXEthief**: extract and decrypt secrets from PXE media.

## References

- [SpecterOps: Misconfiguration Manager, CRED](https://github.com/subat0mik/Misconfiguration-Manager/tree/main/attack-techniques/CRED)
- [GuidePoint: SCCM exploitation, compromising Network Access Accounts](https://www.guidepointsecurity.com/blog/sccm-exploitation-compromising-network-access-accounts/)
- [PXEthief (MWR / Christopher Panayi)](https://github.com/MWR-CyberSec/PXEThief)
