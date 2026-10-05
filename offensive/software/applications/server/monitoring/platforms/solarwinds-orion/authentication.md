---
title: "Authentication: default and weak Orion console credentials"
description: "The SolarWinds Orion web console authenticates with an Admin account (default blank or weak password) and integrates with Active Directory. Default, weak, or reused credentials grant console access, which exposes the managed-device inventory and the stored credentials, and admin access reaches the configuration and features behind the platform's higher-impact issues."
keywords:
  - solarwinds admin
  - orion login
  - default credentials
  - weak password
  - active directory
---

# Authentication

The Orion web console authenticates against its own accounts and, commonly, Active Directory. The built-in `Admin` account historically had a blank or weak default password, and operator-set passwords are frequently weak or reused; AD integration means domain credentials (from elsewhere in an engagement) may also log in. Console access exposes the managed-node inventory, a map of the estate, and the stored device credentials, and administrative access reaches the configuration, the Orion API, and the features that the platform's higher-impact vulnerabilities build on. So default, weak, or reused credentials are a direct route into the platform and, through its stored secrets, into the managed environment.

```bash
# the Orion Admin account (historically blank/weak default)
curl -sk -c cj https://<target>/Orion/Login.aspx --data 'Username=Admin&Password='
# AD-integrated login accepts domain credentials where configured
# spray weak/reused passwords against the console
```

## Exploitation notes

- Try the `Admin` account with a blank and common passwords; the historical default is weak, and operator passwords are often reused.
- AD integration means domain credentials obtained elsewhere may authenticate to Orion; conversely, Orion holds credentials that pivot to the domain, see [Credential harvesting](credential-harvesting.md).
- Console access exposes the node inventory and stored credentials; admin reaches the API and configuration behind the [known exploits](known-exploits.md).
- Where credentials fail, a version-specific [pre-auth exploit](known-exploits.md) may bypass authentication entirely.

## References

- [SolarWinds Orion: accounts](https://documentation.solarwinds.com/)
- [SolarWinds security advisories](https://www.solarwinds.com/trust-center/security-advisories)
