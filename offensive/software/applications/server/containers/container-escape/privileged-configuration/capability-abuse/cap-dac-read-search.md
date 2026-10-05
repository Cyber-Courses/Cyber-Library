---
title: "CAP_DAC_READ_SEARCH: reading any host file with the Shocker technique"
description: "Escaping a container's file isolation with CAP_DAC_READ_SEARCH by using open_by_handle_at to brute-force host inode handles, the Shocker technique, reading any file on the host filesystem including shadow and key material regardless of path."
keywords:
  - CAP_DAC_READ_SEARCH
  - open_by_handle_at
  - Shocker
  - container escape
  - arbitrary file read
---

# CAP_DAC_READ_SEARCH

`CAP_DAC_READ_SEARCH` bypasses file read and directory search permission checks, and it permits `open_by_handle_at`. That syscall opens a file by an opaque handle rather than a path, and handles for the root filesystem are guessable, so a container holding this capability can read host files outside its mount namespace. This is the classic Shocker technique.

```bash
capsh --print | grep -q cap_dac_read_search && echo have
# Shocker: brute-force the host root inode handle, then read a target file
./shocker /etc/shadow        # reads the HOST /etc/shadow via open_by_handle_at
```

## Exploitation notes

- It is a read primitive: no writes, no code execution, but host `/etc/shadow`, SSH keys, and tokens are enough to escalate elsewhere.
- It works because file handles encode the inode on the underlying device, which is shared with the host; the container mount namespace does not gate `open_by_handle_at`.
- For writes rather than reads, see [CAP_DAC_OVERRIDE](cap-dac-override.md).

## References

- [man 2 open_by_handle_at](https://man7.org/linux/man-pages/man2/open_by_handle_at.2.html)
- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
