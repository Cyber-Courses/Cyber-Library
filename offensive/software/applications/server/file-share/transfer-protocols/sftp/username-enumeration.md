---
title: "Username enumeration: discovering valid SFTP users"
description: "Discovering valid usernames on an SFTP server through the SSH authentication behavior, using timing differences and authentication-method responses that distinguish existing from non-existent accounts to build a target list for spraying."
keywords:
  - username enumeration
  - SSH user enumeration
  - timing
  - auth methods
  - account discovery
---

# Username enumeration

Some SSH versions and configurations reveal whether a username exists, through timing differences in password processing or through which authentication methods the server offers for a given user. Enumerating valid accounts narrows a password spray to real targets and avoids noise against non-existent users.

```bash
# Timing/behaviour-based enumeration against vulnerable OpenSSH versions
nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=<user>" <target>
# Public enumeration tooling probes auth responses per username
```

## Exploitation notes

- Enumeration relies on a server that leaks a distinguishable response or timing for valid versus invalid users; patched OpenSSH largely closes timing leaks.
- A confirmed user list makes spraying efficient and quieter.
- Combine with names harvested elsewhere (shares, web, OSINT) rather than pure guessing.

## References

- [HackTricks: pentesting SSH](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ssh.html)
- [OpenSSH](https://www.openssh.com/)
