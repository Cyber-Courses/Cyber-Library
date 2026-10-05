---
title: "Cleartext passwords: capturing the rexec credential"
description: "rexec sends the username and password in cleartext as part of the execution request, so a positioned attacker captures the credential directly from the traffic. The credential is reusable, and because rexec passes it on every command, repeated captures are trivial wherever the service is used."
keywords:
  - cleartext password
  - rexec
  - credential capture
  - sniffing
  - port 512
---

# Cleartext passwords

`rexec` carries the username and password in cleartext within the execution request sent to `rexecd`, so a positioned attacker reads the credential straight off the wire. Because rexec sends the credential with each command invocation (it is not a persistent authenticated session like rlogin but a per-command exchange), every use is another capture opportunity. The captured credential is a real account password, reusable on the host and typically across the legacy environment, obtained with no guessing and nothing logged as a failure.

```bash
# capture the rexec credential from an on-path position
tcpdump -i eth0 -A port 512 -w rexec.pcap
tshark -r rexec.pcap -q -z follow,tcp,ascii,0     # the username/password appear in the request
```

## Exploitation notes

- The credential is in the request itself, so a single captured rexec invocation yields it; no challenge or hashing is involved.
- Per-command transmission means scripted/automated rexec usage (cron jobs, management scripts) leaks the credential repeatedly, making capture easy where rexec is in regular use.
- The captured password is a system account credential, reusable across hosts and sometimes other services; test it broadly.
- Capture is quieter than brute force and avoids any lockout; prefer it where an on-path position exists, and combine with [trusted-host bypass](trusted-host-bypass.md) where trust removes the need for a password entirely.

## References

- [rexecd and the exec protocol](https://linux.die.net/man/8/rexecd)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
