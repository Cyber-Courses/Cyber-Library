---
title: "Cleartext passwords: capturing the rexec credential"
order: 2
description: "rexec sends the username and password in cleartext as part of the execution request, so a positioned attacker captures the credential directly from the traffic. The credential is reusable, and because rexec passes it on every command, repeated captures are trivial wherever the service is used."
keywords:
  - cleartext password
  - rexec
  - credential capture
  - sniffing
  - port 512
---

# Cleartext passwords

`rexec` carries the username and password in cleartext within the execution request sent to `rexecd`, so a positioned attacker reads the credential straight off the wire. Because rexec sends the credential with each command invocation (it is not a persistent authenticated session like rlogin but a per-command exchange), every use is another capture opportunity. The captured password can be tested against the authenticated account on the target within the authorized scope; reuse on other hosts or services exists only where account provisioning or password policy makes it possible.

```bash
# capture the rexec credential from an on-path position
tcpdump -i eth0 -w rexec.pcap port 512
tshark -r rexec.pcap -q -z follow,tcp,ascii,0     # the username/password appear in the request
```

## Exploitation notes

- The credential is in the request itself, so a single captured rexec invocation yields it; no challenge or hashing is involved.
- Per-command transmission means scripted/automated rexec usage (cron jobs, management scripts) leaks the credential repeatedly, making capture easy where rexec is in regular use.
- The captured password may work elsewhere only when credentials are shared or synchronized; do not assume cross-host reuse, and test only systems within the authorized scope.
- Capture is quieter than brute force and avoids failed-login noise; prefer it where an on-path position exists. Do not confuse separately exposed `rsh`/`rlogin` host trust with `rexec` authentication; see the [trusted-host distinction](trusted-host-bypass.md).

## References

- [rexecd and the exec protocol](https://linux.die.net/man/8/rexecd)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
