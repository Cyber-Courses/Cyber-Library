---
title: "gMSA: reading managed service account passwords"
order: 1
description: "Recovering a group Managed Service Account password from Active Directory by reading the msDS-ManagedPassword blob, allowed to the principals in msDS-GroupMSAMembership, and deriving the account's NT hash and AES keys offline."
keywords:
  - gMSA
  - msDS-ManagedPassword
  - ReadGMSAPassword
  - gMSADumper
  - PrincipalsAllowedToRetrieveManagedPassword
---

# gMSA

A group Managed Service Account has its password generated and rotated by the domain and stored in the directory, in the **`msDS-ManagedPassword`** attribute (a blob containing the current and previous passwords). AD only returns that attribute to the principals listed in the account's **`msDS-GroupMSAMembership`** (`PrincipalsAllowedToRetrieveManagedPassword`). So if you control a principal in that set, you read the blob and derive the account's NT hash and Kerberos keys directly from the directory, no code on any host. BloodHound marks this as the `ReadGMSAPassword` edge.

## Reading the password

```bash
# NetExec: dump all gMSA passwords you are allowed to read (uses LDAPS automatically)
nxc ldap <dc> -u user -p pass --gmsa
# authenticate with a machine account's hash where that host may read the gMSA
nxc ldap <dc> -u 'WEB01$' -H <nthash> -d example.local --gmsa

# gMSADumper: read msDS-ManagedPassword and print NTLM / Kerberos keys
gMSADumper.py -u user -p pass -d example.local -l <dc>

# bloodyAD: read the raw attribute
bloodyAD --host <dc> -d example.local -u user -p pass get object 'svc_gmsa$' --attr msDS-ManagedPassword
```

The output is the gMSA's NT hash (and AES keys), which you use with [pass-the-hash](../authentication/ntlm/pass-the-hash.md) or [overpass-the-hash](../authentication/kerberos/pass-the-key-and-overpass-the-hash.md).

## Getting into the read set

The edge you need is write access to the gMSA's `msDS-GroupMSAMembership`, or membership of a group already in it:

- A `GenericWrite`/`GenericAll` ([DACL](../dacl/index.md)) over the gMSA lets you add yourself to `PrincipalsAllowedToRetrieveManagedPassword`, then read.
- Compromise of a **host** already authorised to run the gMSA (its machine account is in the set) lets you read with that machine account.

## Exploitation notes

- gMSA passwords are long and random, so they will not crack; the value is the **hash/keys you read directly**, used as-is.
- A gMSA is frequently **privileged** (it runs a service with real rights), so its hash often grants meaningful access or further edges.
- With the **KDS root key** (readable by Domain Admins / on a DC) you can compute any gMSA's password **offline** for any point in time, which is both an escalation shortcut and gMSA persistence.
- The blob holds the previous password too, useful if rotation happened between recon and use.

## Tools

- **NetExec (`nxc`) `--gmsa`**: read all permitted gMSA passwords from Linux.
- **gMSADumper.py**: read and decode to NTLM/Kerberos keys.
- **bloodyAD** (`get object --attr msDS-ManagedPassword`): raw attribute read, and `set object` on `msDS-GroupMSAMembership` to add yourself.

## References

- [SpecterOps: BloodHound ReadGMSAPassword edge](https://bloodhound.specterops.io/resources/edges/read-gmsa-password)
- [Microsoft (MS-ADTS): msDS-ManagedPassword](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/9cd2fc5e-7305-4fb8-b233-2a60bc3eec68)
- [gMSADumper (micahvandeusen)](https://github.com/micahvandeusen/gMSADumper)
- [NetExec: dumping gMSA over LDAP](https://github.com/Pennyw0rth/NetExec-Wiki/blob/main/ldap-protocol/dump-gmsa.md)
