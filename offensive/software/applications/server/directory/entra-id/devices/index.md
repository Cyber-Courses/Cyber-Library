---
title: "Entra devices"
description: "Abusing Entra device identity: registering or joining rogue devices, bulk-enrollment provisioning tokens, and hybrid-join forgery to obtain primary refresh tokens."
keywords:
  - device registration
  - Azure AD join
  - BPRT
  - hybrid join
  - PRT
---

# Devices

Device identity is a credential in Entra: a registered or joined device gets a **primary refresh token** and can satisfy device-based conditional access. Creating a device identity you control, by registration, bulk-enrollment token, or hybrid-join forgery, yields a PRT and a trusted device, which unlocks SSO and policy-gated apps.

## What folds in here

- **[Device registration](device-registration.md)**: registering a rogue device for a PRT.
- **[BPRT](bprt.md)**: bulk-enrollment provisioning tokens to register at scale.
- **[Hybrid join](hybrid-join.md)**: forging a hybrid-joined device identity.

## References

- [AADInternals: device registration](https://aadinternals.com/aadinternals/)
- [dirkjanm.io: device identity and PRT](https://dirkjanm.io/)
- [ROADtools (roadtx)](https://github.com/dirkjanm/ROADtools)
