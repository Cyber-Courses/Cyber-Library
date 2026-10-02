---
title: "LSASS dumping: extracting live credentials and tickets from memory"
description: "Dumping the LSASS process to recover the NTLM hashes, Kerberos tickets, and sometimes plaintext passwords of every principal with a session on a compromised Windows host."
keywords:
  - LSASS
  - mimikatz
  - sekurlsa
  - credential dumping
  - comsvcs
---

# LSASS dumping

The Local Security Authority Subsystem Service (`lsass.exe`) holds, in memory, the authentication material of every principal currently logged on to the host: NTLM hashes, Kerberos tickets and keys, and (where legacy providers such as WDigest are enabled) plaintext passwords. Dumping LSASS on a host where a privileged user has a session is the classic way to steal their credentials and move laterally as them. It requires local administrator (or `SeDebugPrivilege`) on the target.

## Reading LSASS directly

Mimikatz reads the secrets straight from the live process:

```
privilege::debug
sekurlsa::logonpasswords        # hashes, tickets, and cleartext where present
sekurlsa::ekeys                 # Kerberos encryption keys (AES) for pass-the-key
```

From Linux tooling against the host, NetExec wraps several LSASS methods:

```bash
nxc smb <host> -u admin -p pass -M lsassy       # remote LSASS parse with lsassy
nxc smb <host> -u admin -H <nthash> --sam --lsa
```

## Dump then parse offline

Touching LSASS with a known tool is heavily monitored, so a common pattern is to produce a raw memory dump with a living-off-the-land method and parse it on your own machine:

```
# Built-in comsvcs.dll MiniDump (needs the LSASS PID)
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <lsass-pid> C:\temp\lsass.dmp full

# Then parse offline
pypykatz lsa minidump lsass.dmp
```

Other dumpers (a renamed procdump, nanodump, direct syscall tools) exist precisely to avoid the signatures of the obvious ones.

## What you recover

- **NTLM hashes**: usable immediately for [pass-the-hash](../ntlm/pass-the-hash.md), or cracked offline.
- **Kerberos tickets (TGTs/TGSs)** and **AES keys**: usable for pass-the-ticket and pass-the-key.
- **Plaintext**: only where a provider caches it (WDigest on older or misconfigured systems, some SSO scenarios); not present on a default modern host.

## Exploitation notes

- Target hosts chosen from [session enumeration](../reconnaissance/session-enumeration.md): a host where a Domain Admin is logged on yields a Domain Admin credential.
- On modern Windows, LSASS may run as a protected process (PPL) or behind Credential Guard, which blocks naive reads; bypasses (a PPL-removal driver, or dumping without the standard API) are needed, and Credential Guard removes the plaintext and constrains NTLM reuse.
- Prefer AES keys over the NTLM hash where available, since AES pass-the-key avoids the weaker RC4 path that some environments monitor or restrict.

## Tools

- **Mimikatz** (`sekurlsa::logonpasswords`, `sekurlsa::ekeys`): the reference live reader.
- **pypykatz**: offline minidump parsing, and remote LSASS parsing.
- **nanodump / comsvcs MiniDump**: low-signature dumping to parse offline.
- **NetExec (nxc) -M lsassy**: remote LSASS extraction.

## References

- The Hacker Recipes: LSASS secrets
- Microsoft: LSA and credential protection (Credential Guard, PPL)
