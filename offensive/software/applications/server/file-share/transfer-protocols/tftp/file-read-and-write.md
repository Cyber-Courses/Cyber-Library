---
title: "File read and write: unauthenticated TFTP transfers"
description: "Reading and writing files on a TFTP server with no authentication, using GET to pull known filenames such as network-device configurations and firmware, and PUT to upload where the server allows writes, since TFTP has no access control."
keywords:
  - TFTP GET
  - TFTP PUT
  - router config
  - firmware
  - unauthenticated
---

# File read and write

TFTP has no authentication and no directory listing, so access is by exact filename. GET retrieves known files, and on write-enabled servers PUT uploads them. The highest-value reads are network-device configurations and firmware, which TFTP is commonly used to serve and which routinely embed credentials and SNMP strings.

```bash
tftp <target>
tftp> get running-config                      # or startup-config, <hostname>-confg
tftp> get pxelinux.cfg/default                 # PXE boot config
tftp> put shell                                # if writes are allowed
# Scripted
curl -o config tftp://<target>/running-config
```

## Exploitation notes

- Guess config filenames by convention: `running-config`, `startup-config`, `<hostname>-confg`, and vendor-specific names; device configs hold enable passwords and SNMP communities.
- PXE environments expose `pxelinux.cfg/` and kickstart or preseed files that can contain install-time credentials.
- Write access lets you tamper with a config or firmware a device will fetch, or stage a file for another service.

## References

- [HackTricks: pentesting TFTP](https://book.hacktricks.wiki/en/network-services-pentesting/69-udp-tftp.html)
- [RFC 1350: TFTP](https://www.rfc-editor.org/rfc/rfc1350)
