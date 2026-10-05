---
title: "Banner grabbing: SSH version and implementation disclosure"
description: "An SSH server announces its protocol version and software in a cleartext banner before authentication, for example SSH-2.0-OpenSSH_8.9p1 Ubuntu. That string identifies the implementation, version, and often the OS/distribution, mapping the service directly to its known vulnerabilities and shaping the user-enumeration and exploit choices."
keywords:
  - ssh banner
  - version detection
  - openssh version
  - fingerprint
  - identification
---

# Banner grabbing

The first thing an SSH server sends, before any authentication and in cleartext, is its identification banner: a line like `SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4`. That single string gives the protocol version, the implementation (OpenSSH, Dropbear, libssh, a vendor stack), the exact software version, and frequently the OS or distribution packaging. It is the anchor for everything else: the version maps to the implementation's known vulnerabilities, distinguishes OpenSSH from Dropbear (common on embedded devices, with its own bugs), and tells you which user-enumeration and exploit techniques apply to this build.

```bash
# grab the banner several ways
nc <target> 22                                 # prints SSH-2.0-...
echo | nc <target> 22 | head -1
nmap -p22 -sV <target>                         # service/version with OS hints
# at scale across a range
for h in $(cat hosts.txt); do echo "$h $(echo | nc -w2 $h 22 | head -1)"; done
```

## Exploitation notes

- The implementation and version are the key facts: OpenSSH, Dropbear, libssh, and vendor forks have distinct vulnerability histories, and the exact version maps to specific advisories (user-enumeration flaws, auth bypasses, memory-corruption bugs).
- Distribution strings (Ubuntu, Debian backport suffixes) refine the patch level, since distributions backport fixes without changing the upstream version number.
- Dropbear banners indicate an embedded/IoT device, pointing toward [default credentials](../authentication/default-credentials.md) and device-specific issues.
- The banner is cleartext and pre-auth, so this costs nothing and is the first step; feed the version into a vulnerability lookup and the implementation into the right user-enumeration method.

## References

- [HackTricks: SSH banner](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [OpenSSH release notes (version mapping)](https://www.openssh.com/releasenotes.html)
