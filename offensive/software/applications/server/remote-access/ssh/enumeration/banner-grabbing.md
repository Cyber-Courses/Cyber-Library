---
title: "Banner grabbing: SSH version and implementation disclosure"
description: "An SSH server sends a cleartext identification string before key exchange. It can suggest the protocol, implementation, and package lineage, but it is self-reported and does not by itself establish the installed build, operating system, or patch status."
keywords:
  - ssh banner
  - version detection
  - openssh version
  - fingerprint
  - identification
---

# Banner grabbing

Before key exchange, an SSH server sends a cleartext identification string such as `SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4`. The protocol field is meaningful, while the software comment is implementation-controlled: it may be customized, suppressed, vendor-modified, or associated with a distribution that backports fixes. Treat it as a fingerprinting hint. Confirm the actual package build and compare it with the applicable vendor advisory before assigning vulnerability status.

```bash
# grab the banner several ways
nc <target> 22                                 # prints SSH-2.0-...
echo | nc <target> 22 | head -1
nmap -p22 -sV <target>                         # service/version with OS hints
# at scale across a range
for h in $(cat hosts.txt); do echo "$h $(echo | nc -w2 $h 22 | head -1)"; done
```

## Exploitation notes

- The string can distinguish likely implementations such as OpenSSH, Dropbear, or libssh, but compatible and vendor-modified stacks may present custom values.
- Distribution suffixes can identify package lineage; use the distribution's package changelog or security advisory because fixes may be backported without changing the upstream version component.
- A Dropbear banner identifies the SSH implementation, not the device category, firmware state, or presence of default credentials.
- Use the banner to choose what to verify, not as sole evidence that a CVE applies.

## References

- [OpenSSH release notes (version mapping)](https://www.openssh.com/releasenotes.html)
- [OpenSSH portable notes on vendor modifications](https://www.openssh.com/portable.html)
