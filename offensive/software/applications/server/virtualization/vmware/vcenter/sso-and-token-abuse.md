---
title: "SSO and token abuse: forging vCenter identity through the SSO system"
description: "vCenter Single Sign-On issues SAML tokens backed by a signing certificate held in the vmdir/VMware Directory and the STS. An attacker who reaches that key material, or the vpxuser and solution-user credentials in the vCenter database, forges or reuses tokens to authenticate as administrator, obtaining full vСenter control without a password."
keywords:
  - vcenter sso
  - saml
  - sts
  - vmdir
  - vpxuser
---

# SSO and token abuse

vCenter authentication runs through Single Sign-On: the Security Token Service (STS) issues SAML tokens signed by a certificate whose private key lives in the embedded VMware Directory (vmdir) and the STS configuration. Services and administrators authenticate by presenting these tokens. The abuse is to obtain the signing key or stored service credentials and then forge or reuse tokens, authenticating as `administrator@vsphere.local` or a solution user without ever knowing a password. The relevant secrets sit on the vCenter appliance and in its databases.

## Where the key material lives

```bash
# on the vCenter appliance: the STS/IdP signing key and certs
ls -l /etc/vmware-sso/ /storage/db/vmware-vmdir/ 2>/dev/null
# the IdP signer certificate and key used to sign SAML tokens
/usr/lib/vmware-vmafd/bin/vecs-cli entry list --store STS_INTERNAL_SSL_CERT 2>/dev/null
# vmdir holds identity data; it is an LDAP directory under /storage/db/vmware-vmdir
```

## Forge and reuse tokens

```bash
# with the STS IdP signing key, mint a SAML assertion for a privileged subject
# (tools automate building a valid, signed SAML token for administrator@vsphere.local),
# then present it to the vSphere API to obtain an authenticated session
# the vpxuser credential (vCenter -> ESXi host auth) is stored in the vCenter DB,
# encrypted with a key also on the appliance; recovering it authenticates to hosts
```

The `vpxuser` account is how vCenter authenticates to each managed ESXi host; its credential is kept in the vCenter database, encrypted with a key present on the appliance, so an attacker with appliance access recovers it and then logs in to every managed host directly.

## Exploitation notes

- Appliance file access (from a management exploit or stolen root) is the usual precondition; from there the STS signing key enables forging admin SAML tokens, bypassing password authentication entirely.
- Forged SAML tokens are accepted by the API as a valid SSO session; tools exist that assemble and sign the assertion given the IdP key, yielding an administrator session.
- `vpxuser` recovery pivots from vCenter to every ESXi host it manages, so SSO compromise cascades to the whole fleet; combine with [ESXi host access](../esxi/host-access-and-shell.md).
- Solution users (the internal service identities) are similarly backed by certificates in vmdir; recovering them authenticates as trusted vCenter services.

## Tools

- [vCenter SAML token tooling (e.g. vcenter_saml_login research)](https://github.com/horizon3ai)

## References

- [VMware: vCenter Single Sign-On architecture](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [Horizon3: vCenter SSO/SAML research](https://www.horizon3.ai/attack-research/)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
