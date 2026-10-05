---
title: "Null session: anonymous access to SMB shares and RPC"
description: "A null session is an unauthenticated SMB connection (empty username and password) that, where permitted, grants access to the IPC$ pipe and sometimes shares. Through IPC$ it reaches the RPC interfaces that enumerate users, groups, shares, and the password policy, making an allowed null session a rich pre-credential reconnaissance foothold."
keywords:
  - null session
  - ipc$
  - rpcclient
  - anonymous
  - enumeration
---

# Null session

A null session is an SMB connection made with an empty username and password. Historically Windows allowed null sessions broad access; modern systems restrict them, but they are still frequently permitted to the `IPC$` share (the named-pipe endpoint), and misconfigured servers and many Samba deployments allow more. The value is what `IPC$` exposes: the MS-RPC interfaces reachable over named pipes, through which an unauthenticated client enumerates users, groups, shares, and the account policy, reconnaissance that normally needs credentials.

```bash
# test and use a null session
nxc smb <target> -u '' -p '' --shares           # anonymous share access
rpcclient -U '' -N <target>                      # null RPC session
#   rpcclient $> srvinfo            # server type/version
#   rpcclient $> enumdomusers       # domain/local users (SID + name)
#   rpcclient $> enumdomgroups      # groups
#   rpcclient $> querydominfo       # domain info
#   rpcclient $> getdompwinfo       # password policy (lockout, min length)
#   rpcclient $> lsaquery           # domain SID
```

## RID cycling

Even when `enumdomusers` is blocked, a null (or guest) session that can resolve SIDs enumerates users by cycling relative identifiers against the domain SID:

```bash
# brute-force RIDs to recover usernames when direct enumeration is denied
nxc smb <target> -u '' -p '' --rid-brute
# or via rpcclient: lookupsids S-1-5-21-...-500, -501, -1000, -1001, ...
lookupnames <sid>   # map names to SIDs and back within rpcclient
```

RID cycling works because account SIDs are the domain SID plus a sequential RID (500 = Administrator, 501 = Guest, 1000+ = created accounts), so resolving each RID recovers the name even without a listing right.

## Exploitation notes

- The recovered user list feeds password spraying and AS-REP/Kerberoast targeting; the password policy (from `getdompwinfo`) sets a safe spray threshold to avoid lockout.
- Null-session access to `IPC$` is common even where data shares are locked down, so always test it; Samba servers often permit more than Windows.
- RID cycling is the fallback when `enumdomusers` is restricted but SID lookups are allowed; `--rid-brute` automates it.
- A null session is reconnaissance, not access to data; combine the user list and policy with share enumeration and the domain attack paths.

## Tools

- [rpcclient (Samba)](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html)
- [NetExec](https://github.com/Pennyw0rth/NetExec)
- [enum4linux-ng](https://github.com/cddmp/enum4linux-ng)

## References

- [NetExec: SMB enumeration](https://www.netexec.wiki/smb-protocol/enumeration)
- [MS-SRVS and MS-SAMR (RPC over IPC$)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-samr/)
