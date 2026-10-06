---
title: "Primary refresh token: stealing and forging the PRT"
description: "Stealing and replaying the primary refresh token: extracting the PRT and session key from a joined device and forging PRT cookies with ROADtoken and AADInternals."
keywords:
  - primary refresh token
  - PRT
  - PRT cookie
  - ROADtoken
  - AADInternals
---

# Primary refresh token

The **primary refresh token** is the device-bound credential that gives Entra single sign-on on a joined machine. With local access to the device you extract the PRT and its session key, then forge **PRT cookies** (`x-ms-RefreshTokenCredential`) that authenticate to any Entra app as the user, including through MFA, because the PRT already carries the strong-auth claim.

## Extract and forge

```powershell
# AADInternals: pull the PRT and session key from the signed-in device
Get-AADIntUserPRTToken
# or derive a PRT from stolen key material
Export-AADIntLocalDeviceCertificate ; New-AADIntUserPRTToken

# ROADtoken: generate a PRT cookie usable in a browser
roadtoken -a <prt> --derived-sessionkey <key>
```

Drop the resulting cookie into the `login.microsoftonline.com` request as `x-ms-RefreshTokenCredential` and you are signed in as the user.

## Exploitation notes

- The PRT satisfies MFA on its own, so a PRT cookie reaches conditional-access-gated apps without a second factor.
- PRTs are device-bound; you need the device's transport key, which is why local admin or a token-broker hook on the box is the prerequisite.
- A forged PRT cookie is a durable SSO foothold until the PRT is revoked or the device is disabled.

## Tools

- **AADInternals** (`Get-AADIntUserPRTToken`): extract and mint PRT tokens.
- **ROADtoken** / **roadtx** (ROADtools): forge PRT cookies.

## References

- [dirkjanm.io: digging into the PRT](https://dirkjanm.io/)
- [AADInternals: PRT](https://aadinternals.com/aadinternals/)
- [ROADtools (roadtx)](https://github.com/dirkjanm/ROADtools)
