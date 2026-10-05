---
title: "SSO and token abuse: forging vCenter administrator access"
description: "Abusing the VMware vCenter single sign-on system to gain administrator: recovering the SSO signing key material from the vmdir identity store and IDP configuration to forge SAML assertions for an administrator, the vSphere equivalent of a golden SAML attack."
keywords:
  - vCenter SSO
  - SAML
  - vmdir
  - golden SAML
  - token forgery
---

# SSO and token abuse

vCenter authentication runs through SSO, which issues SAML tokens signed by the IDP's private key. An attacker who reaches the appliance filesystem or the vmdir identity store can recover that signing key and the IDP configuration, then forge a SAML assertion for the SSO `administrator`, authenticating as full admin without a password. This is the vSphere form of a golden SAML attack.

```bash
# On a compromised vCenter appliance: the SSO IDP signing key and vmdir data
ls /storage/db/vmware-vmdir/     # vmdir database (data.mdb)
# Extract the IDP signing certificate/key, then mint a SAML token for administrator@vsphere.local
```

## Exploitation notes

- Forged SSO tokens grant `administrator@vsphere.local`, which is control of every managed host and VM, and they are not tied to a password, so password resets do not revoke them.
- The signing key is recovered from the appliance, so this follows an initial foothold on vCenter, often via a [Known management exploit](known-management-exploits.md).
- The vCenter database (not vmdir) holds the per-host `vpxuser` passwords vCenter uses to manage each ESXi host, so the same appliance compromise is a direct pivot to every host shell.

## References

- [VMware: vCenter Single Sign-On](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-authentication/GUID-D15C8E8C-6D5F-4F7B-9B3E-9F8B9E8D1234.html)
- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
