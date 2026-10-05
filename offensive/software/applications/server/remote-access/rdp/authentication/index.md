---
title: "Authentication: attacking RDP login"
description: "RDP authenticates with Windows credentials, so it is a brute-force, password-spray, and credential-stuffing target, and a prime landing point for reused and breached passwords. Network Level Authentication changes where credentials are checked but not whether they can be guessed. Valid RDP credentials give an interactive desktop, often with local or domain privilege."
keywords:
  - rdp authentication
  - brute force
  - password spray
  - credential stuffing
  - nla
---

# Authentication

RDP logs in with Windows credentials (local or domain), which makes it a direct target for guessing and reuse attacks and one of the most common ways stolen or sprayed credentials turn into interactive access. Brute force and spraying work against the logon, credential stuffing replays breached username/password pairs, and Network Level Authentication (NLA) only moves the check earlier (into CredSSP) rather than preventing guessing. A valid credential yields a full interactive desktop, frequently with administrative or domain rights, so RDP authentication is a high-value, high-frequency attack surface, subject to Windows account lockout.

```bash
# spray/guess against RDP, respecting lockout (domain\user forms from enumeration)
nxc rdp <target> -u users.txt -p 'Winter2025!'          # marks valid + whether admin
hydra -L users.txt -p 'Winter2025!' rdp://<target> -t 1
```

## Subtopics

- **[Password brute force](password-brute-force.md)**: online guessing against the logon.
- **[Credential stuffing](credential-stuffing.md)**: replaying breached and reused credentials.
- **[NLA bypass](nla-bypass.md)**: Network Level Authentication considerations and misconfigurations.

## References

- [HackTricks: RDP authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
- [Microsoft: RDP security](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
