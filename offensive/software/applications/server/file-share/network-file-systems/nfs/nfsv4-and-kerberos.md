---
title: "NFSv4 and Kerberos: the stronger configuration and its weak points"
description: "NFSv4 adds a single pseudo-filesystem, name-based identity mapping, and optional Kerberos security (sec=krb5). Kerberos authenticates the user and, with krb5i/krb5p, protects integrity and privacy, defeating UID spoofing. The weak points are servers that still allow sec=sys fallback, misconfigured id mapping, and reliance on stolen Kerberos tickets or keytabs."
keywords:
  - nfsv4
  - sec=krb5
  - idmapd
  - keytab
  - kerberos
---

# NFSv4 and Kerberos

NFSv4 modernises the protocol: a single exported pseudo-filesystem instead of per-export mounts, name-based identity mapping (`user@domain` resolved by `idmapd`) rather than raw numeric UIDs, and optional Kerberos security. With `sec=krb5` the user is authenticated by a Kerberos ticket, and `krb5i` and `krb5p` add integrity and privacy protection, which defeats the `AUTH_SYS` UID-spoofing attacks because identity is now cryptographically established rather than asserted. The offensive interest is therefore in the configurations that fall short of this and in abusing Kerberos credentials rather than forging UIDs.

## Weak points

```bash
# does the server still allow sec=sys (AUTH_SYS) alongside or instead of krb5?
mount -t nfs -o vers=4,sec=sys <target>:/ /mnt/nfs   # if this succeeds, spoofing applies
# what security flavours are offered?
nmap -p2049 --script nfs-showmount <target>; rpcinfo -p <target>
```

- **sec=sys fallback**: many deployments enable Kerberos but still accept `sec=sys`, so an attacker simply mounts with `sec=sys` and the [UID/GID spoofing](uid-and-gid-spoofing.md) attacks apply unchanged. This is the most common real-world gap.
- **id mapping misconfiguration**: if `idmapd` domains or mappings are inconsistent, identities may collapse to `nobody` or map unexpectedly, sometimes widening access.
- **stolen Kerberos credentials**: where `krb5` is enforced, access needs a valid ticket; the attack shifts to obtaining a user's TGT/service ticket or a host keytab (`/etc/krb5.keytab`) and using it to mount as that principal.

```bash
# with a stolen keytab or ticket, authenticate then mount as that principal
kinit -kt /loot/krb5.keytab host/server@REALM
mount -t nfs -o vers=4,sec=krb5 <target>:/ /mnt/nfs   # access as the authenticated identity
```

## Exploitation notes

- Always test `sec=sys` first: an enforced-Kerberos server that still permits AUTH_SYS fallback is fully exposed to UID spoofing, which is a frequent misconfiguration.
- Under enforced `krb5`, the attack is credential theft, not forgery: a captured keytab authenticates as the host or service principal; a stolen TGT (from a compromised client) authenticates as the user.
- `krb5p` encrypts the traffic, so passive capture of NFS data is defeated; `krb5` (auth only) still leaves data on the wire readable.
- The id-mapping domain must match between client and server for names to resolve; mismatches are a source of both denial and, occasionally, over-broad mapping.

## References

- [RFC 8881: NFSv4.1 security](https://datatracker.ietf.org/doc/html/rfc8881)
- [nfs(5): sec= options](https://man7.org/linux/man-pages/man5/nfs.5.html)
- [Linux idmapd](https://man7.org/linux/man-pages/man8/rpc.idmapd.8.html)
