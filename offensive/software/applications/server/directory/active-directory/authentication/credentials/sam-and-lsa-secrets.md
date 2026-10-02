---
title: "SAM and LSA secrets: local hashes and cached machine and service secrets"
description: "Dumping the local SAM database for local-account hashes and the LSA secrets for cached domain credentials, service-account passwords, and the machine account's secret from a host's registry hives."
keywords:
  - SAM
  - LSA secrets
  - secretsdump
  - machine account
  - cached credentials
---

# SAM and LSA secrets

Beyond live memory, a Windows host keeps credential material in the registry: the **SAM** holds local-account password hashes, and the **LSA secrets** hold cached domain logons, service-account passwords, and the host's own machine-account secret. These are readable with local admin (SYSTEM) and, unlike LSASS, do not require the target principals to be currently logged on.

## Dumping the hives

The secrets live in the `SAM`, `SYSTEM`, and `SECURITY` registry hives. The standard remote method pulls them over SMB:

```bash
# Impacket: dump SAM + LSA + (if DC) NTDS over the network
secretsdump.py 'EXAMPLE/admin:password@<host>'
secretsdump.py -hashes :<nthash> 'EXAMPLE/admin@<host>'

# NetExec
nxc smb <host> -u admin -p pass --sam --lsa
```

Locally, save the hives and parse offline:

```
reg save HKLM\SAM sam.save & reg save HKLM\SYSTEM system.save & reg save HKLM\SECURITY security.save
secretsdump.py -sam sam.save -system system.save -security security.save LOCAL
```

## What each store yields

- **SAM**: NTLM hashes of **local** accounts (including the local Administrator). A reused local admin password across machines enables lateral movement by [pass-the-hash](../ntlm/pass-the-hash.md); where LAPS randomizes it, each host differs.
- **LSA secrets** (`SECURITY` hive) hold several high-value items:
  - **`$MACHINE.ACC`**: the machine account's password/hash, which authenticates as the computer (useful for RBCD, S4U, and silver tickets against that host's services).
  - **Cached domain logons** (`CACHEDDUMP`, the MSCache2/DCC2 format): the last interactive domain users to log on, crackable offline but not usable directly for pass-the-hash.
  - **Service account secrets**: the plaintext passwords of services configured to run as a domain account, stored so the service can start.
  - **DPAPI master key backup** and auto-logon passwords (`DefaultPassword`).

## Exploitation notes

- The machine-account secret is often overlooked but powerful: a computer account can be the pivot for resource-based constrained delegation and silver tickets.
- DCC2 (cached domain) hashes use a slow, salted format, so they crack far more slowly than NTLM; prioritize other material unless the password is weak.
- Service-account plaintext from LSA secrets is immediately usable and frequently belongs to a privileged account.

## Tools

- **secretsdump.py** (Impacket): remote and offline SAM/LSA/NTDS extraction.
- **NetExec (nxc) --sam --lsa**: fast SAM and LSA dumps across hosts.
- **Mimikatz** (`lsadump::sam`, `lsadump::secrets`): on-host dumps.

## References

- The Hacker Recipes: SAM and LSA secrets
- Microsoft: LSA secrets and cached credentials
