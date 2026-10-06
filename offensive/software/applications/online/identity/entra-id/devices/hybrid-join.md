---
title: "Hybrid join: forging a hybrid-joined device identity"
description: "Forging a hybrid Azure AD join: writing device objects or abusing on-prem sync to create a trusted hybrid-joined device identity."
keywords:
  - hybrid join
  - hybrid Azure AD join
  - device object
  - on-prem sync
  - PRT
---

# Hybrid join

A hybrid-joined device is an on-prem Active Directory computer that is also registered in Entra. With control of on-prem AD (or the sync path), you forge the pieces of a hybrid join, writing the device's `userCertificate` attribute and letting Entra Connect sync it, to create a trusted hybrid-joined identity that yields a primary refresh token.

## Forge the join

```powershell
# write a self-signed device certificate to the on-prem computer object, then let sync carry it to Entra
# AADInternals handles the device cert and PRT once the object is synced
Set-AADIntDeviceRegKeys ; Get-AADIntUserPRTToken
```

See [hybrid identity](../../../../server/directory/active-directory/trusts/entra-hybrid.md) for the Entra Connect sync internals this abuses.

## Exploitation notes

- This bridges an on-prem AD compromise into Entra: a hybrid device identity is trusted by cloud conditional access and yields a cloud PRT.
- It requires write access to the on-prem computer object (or the sync account), so it is an on-prem-to-cloud pivot, not an external technique.
- The forged device looks legitimately hybrid-joined, making it durable and low-signal.

## Tools

- **AADInternals**: device certificate and PRT handling.
- **ROADtools** (`roadtx`): device and PRT operations.

## References

- [dirkjanm.io: hybrid join and PRTs](https://dirkjanm.io/)
- [AADInternals: hybrid join](https://aadinternals.com/aadinternals/)
- [HackTricks Cloud: hybrid identity](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
