---
title: "Null session: anonymous SMB access and enumeration"
description: "Abusing an SMB null session, an unauthenticated connection to the IPC share, to enumerate users, groups, shares, and the password policy on legacy or misconfigured Windows and Samba servers that allow anonymous access."
keywords:
  - null session
  - IPC$
  - anonymous SMB
  - enum4linux
  - rpcclient
---

# Null session

A null session is an SMB connection with an empty username and password to the `IPC$` share. On legacy Windows and misconfigured Samba, it exposes a surprising amount through the RPC interfaces: users, groups, shares, the password policy, and domain information, all without credentials. It is the classic anonymous enumeration foothold.

```bash
# Enumerate everything reachable anonymously
enum4linux-ng -A <target>
rpcclient -U '' -N <target> -c 'enumdomusers; querydominfo; getdompwinfo'
nxc smb <target> -u '' -p '' --users --shares --pass-pol
```

## Exploitation notes

- User lists feed password spraying; the password policy tells you the lockout threshold so the spray stays safe.
- Modern Windows restricts anonymous RPC (`RestrictAnonymous`), so null sessions mostly succeed on older systems and Samba; a guest session is the modern equivalent.
- SIDs enumerated here support RID cycling to recover more usernames.

## References

- [enum4linux-ng](https://github.com/cddmp/enum4linux-ng)
- [HackTricks: pentesting SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
