---
title: "Pass-the-hash: authenticating with the NT hash alone"
order: 4
description: "Authenticating to Windows services directly with a stolen NT hash instead of a password, and executing remotely over SMB, WMI, and WinRM, because NTLM derives its response from the hash and never needs the plaintext."
keywords:
  - pass the hash
  - NT hash
  - overpass the hash
  - lateral movement
  - NTLM
---

# Pass-the-hash

The NTLM response is computed from the NT hash, not the password, so the **hash is the credential**. If you recover an account's NT hash ([LSASS](../credentials/lsass-dumping.md), [SAM](../credentials/sam-and-lsa-secrets.md), or [NTDS/DCSync](../credentials/ntds-and-dcsync.md)), you can authenticate as that account anywhere NTLM is accepted without ever cracking it. This is the primary lateral-movement technique on NTLM.

**Lineage.** The flaw is as old as the protocol: the NTLM response is derived directly from the NT hash, so the hash itself is a sufficient credential, and LanMan's weak DES-based hashing made early Windows secrets cheap to crack on top of that. Hash reuse became practical in the late 1990s and a standard toolkit through the 2000s. In-memory credential theft industrialised it, and Restricted Admin mode, Protected Users, and Credential Guard are the later answers that push today's operators toward AES overpass-the-hash and Kerberos pivots where the raw NT hash is refused.

## Executing with a hash

```bash
# Impacket: remote exec authenticating with the NT hash
psexec.py -hashes :<nthash> EXAMPLE/admin@<host>      # SMB service, SYSTEM
wmiexec.py -hashes :<nthash> EXAMPLE/admin@<host>     # WMI, quieter
smbexec.py -hashes :<nthash> EXAMPLE/admin@<host>

# NetExec: spray a hash across many hosts to map local-admin access
nxc smb <subnet> -u admin -H <nthash> --local-auth
nxc smb <subnet> -u admin -H <nthash>                 # domain account

# Evil-WinRM over WinRM
evil-winrm -i <host> -u admin -H <nthash>
```

The `-H` / `-hashes` sweep across a subnet is how you find **where** a recovered hash grants local admin, turning one dumped hash into a map of reachable hosts.

## Local vs domain accounts

- A **local** account's hash (from SAM) works on every machine that shares that password, the classic local-admin reuse problem; LAPS breaks it by randomizing per host.
- A **domain** account's hash works anywhere the account has rights, so a dumped domain-admin hash is immediate domain-wide access.

## Overpass-the-hash (hash to Kerberos)

NTLM pass-the-hash can be noisy or blocked where only Kerberos is accepted. **Overpass-the-hash** uses the NT hash (or AES key) to request a Kerberos TGT, converting the hash into a ticket and letting you move with Kerberos instead:

```bash
getTGT.py -hashes :<nthash> EXAMPLE/admin        # request a TGT from the hash
# then export KRB5CCNAME and use -k with the Impacket exec tools
```

Using the **AES key** instead of the RC4/NT hash for this is stealthier where RC4 is monitored or disabled. This bridges to the Kerberos techniques covered in the Kerberos section.

## Exploitation notes

- Pass-the-hash needs no cracking, so a strong, uncrackable password is no defense once its hash is recovered; only rotating the password (or a different account) invalidates the hash.
- The built-in RID-500 Administrator is the highest-value local target because its SAM hash is present on every host and often reused.
- Where only AES Kerberos is accepted, use **pass-the-key** (the AES key) rather than the NT hash; recover AES keys from LSASS or an `-just-dc` NTDS dump.

## Tools

- **Impacket** (`psexec.py`, `wmiexec.py`, `smbexec.py`, `getTGT.py`): hash-based exec and overpass-the-hash.
- **NetExec (nxc)**: hash spraying and access mapping across hosts.
- **Evil-WinRM / Mimikatz `sekurlsa::pth`**: WinRM exec and on-host hash injection.

## References

- [The Hacker Recipes: pass the hash](https://www.thehacker.recipes/ad/movement/ntlm/pth)
- [Impacket (fortra): psexec/wmiexec/smbexec and -hashes](https://github.com/fortra/impacket)
- [NetExec: authenticating with -H across protocols](https://www.netexec.wiki/)
- [Microsoft (MS-NLMP): NTLM authentication protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/b38c36ed-2804-4868-a9ff-8dd3182128e4)
