---
title: "BPRT: bulk-enrollment tokens to register devices at scale"
description: "Abusing a bulk-enrollment provisioning token to register devices and mint primary refresh tokens at scale."
keywords:
  - BPRT
  - bulk enrollment
  - provisioning token
  - device registration
  - PRT
---

# BPRT

A bulk-enrollment provisioning token (BPRT) is a reusable credential meant for provisioning many devices at once. Obtained or forged, it lets you register devices without a per-user token and mint primary refresh tokens at scale, a powerful persistence and impersonation primitive.

## Create and use a BPRT

```powershell
# AADInternals: create a BPRT (needs an account allowed to bulk-enrol), then use it
$bprt = New-AADIntBulkPRTToken -Name "kiosk"
# register a device with the BPRT and obtain a PRT
Join-AADIntDeviceToAzureAD -DeviceName "kiosk-07" -BPRT $bprt
```

## Exploitation notes

- A BPRT is not bound to one user, so it registers arbitrary devices and is reusable until it expires or is revoked, excellent persistence.
- Devices created through a BPRT appear as legitimately provisioned, blending into normal enrollment.
- Creating a BPRT requires an account with bulk-enrollment rights, so it is typically a post-escalation persistence step.

## Tools

- **AADInternals** (`New-AADIntBulkPRTToken`).

## References

- [AADInternals: BPRT](https://aadinternals.com/aadinternals/)
- [dirkjanm.io: device identity](https://dirkjanm.io/)
- [HackTricks Cloud: device abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
