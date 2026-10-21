---
title: "Authentication: attacking RDP login"
description: "RDP authenticates with Windows credentials, so it is a target for guessing and credential reuse. Network Level Authentication changes when credentials are checked but does not prevent guessing. A successful login has the authenticated account's existing rights; administrative access and credential reuse must be established separately."
keywords:
  - rdp authentication
  - brute force
  - password spray
  - credential stuffing
  - nla
---

# Authentication

RDP logs in with Windows credentials (local or domain), which makes it a direct target for guessing and reuse attacks. Brute force and spraying work against the logon, credential stuffing replays breached username/password pairs, and Network Level Authentication (NLA) moves the check earlier into CredSSP rather than preventing guessing. A successful login creates an interactive session under that account's existing local and domain permissions. Ordinary members of Remote Desktop Users can be authorized to connect without being administrators, so administrative rights and reuse on other systems must be tested separately.

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
