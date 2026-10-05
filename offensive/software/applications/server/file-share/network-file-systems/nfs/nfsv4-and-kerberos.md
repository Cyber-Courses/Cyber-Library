---
title: "NFSv4 and Kerberos: attacking the modern NFS model"
description: "Attacking NFSv4 specifics: its single-port model and name-based identities mapped by idmapd, and the security flavors from weak AUTH_SYS to Kerberos (krb5, krb5i, krb5p), including forcing a downgrade from Kerberos to AUTH_SYS where the server still allows it."
keywords:
  - NFSv4
  - idmapd
  - sec=krb5
  - AUTH_SYS downgrade
  - Kerberos
---

# NFSv4 and Kerberos

NFSv4 changed the model: it uses a single port (2049, no separate portmapper), string-based user and group names mapped by `idmapd`, and a per-export security flavor. With `sec=sys` (AUTH_SYS) it is still the trust-the-client model, vulnerable to UID spoofing. With `sec=krb5` it requires Kerberos tickets and authenticates the user; `krb5i` adds integrity and `krb5p` encryption.

```bash
# NFSv4 mounts on 2049 directly; check the server's accepted security flavors
mount -t nfs4 -o sec=sys <target>:/ /mnt/nfs     # if sec=sys is allowed, UID spoofing applies
showmount -e <target> 2>/dev/null                # v3 compatibility may still list exports
```

## Exploitation notes

- If an export lists multiple flavors (`sys` and `krb5`), a client can often choose `sec=sys` and bypass Kerberos entirely; this downgrade is the key attack.
- Under real Kerberos (`krb5`), access needs a valid ticket, so the attack shifts to obtaining one (keytabs, ticket theft) rather than UID tricks.
- `idmapd` name mapping means owners appear as `user@domain`; mismatched domains can cause files to map to `nobody`, a hint the export expects a specific realm.

## References

- [man 5 nfs](https://man7.org/linux/man-pages/man5/nfs.5.html)
- [RFC 7530: NFSv4](https://www.rfc-editor.org/rfc/rfc7530)
