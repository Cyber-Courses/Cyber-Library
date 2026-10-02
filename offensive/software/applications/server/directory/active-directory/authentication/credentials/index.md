---
title: "Credentials: dumping, cracking, and guessing Active Directory secrets"
description: "Obtaining credential material in Active Directory: dumping secrets from LSASS, the SAM, DPAPI, and the NTDS database (including DCSync), cracking the recovered hashes offline, and guessing passwords through spraying."
keywords:
  - credential dumping
  - LSASS
  - NTDS
  - DCSync
  - password spraying
---

# Credentials

Credentials are the currency of an Active Directory attack. This section covers the three ways to acquire them: **dumping** secrets already present on a host or in the directory, **cracking** the recovered hashes offline into usable passwords, and **guessing** them against the domain through spraying. Everything else in the authentication and credentials section either consumes these secrets or produces more of them.

## Where secrets live

A compromised Windows host and the directory itself hold several credential stores:

- **LSASS memory**: the plaintext, NTLM hashes, and Kerberos tickets of every principal with a session on the host.
- **The SAM and LSA secrets**: local account hashes and cached service-account and machine-account secrets in the registry.
- **DPAPI-protected stores**: saved browser and application passwords, and credentials in the Windows vault.
- **The NTDS database**: on a domain controller (or via DCSync from any host with the replication right), the NTLM hashes of every account in the domain, including `krbtgt`.
- **The directory**: passwords in attributes, LAPS and gMSA managed passwords (covered under [credentials in attributes](../../../ldap/credentials-in-attributes.md)).

## Pages

- **[LSASS dumping](lsass-dumping.md)**: extracting live credentials and tickets from LSASS memory.
- **[SAM and LSA secrets](sam-and-lsa-secrets.md)**: local hashes and cached machine and service secrets.
- **[DPAPI secrets](dpapi-secrets.md)**: browser, vault, and application credentials.
- **[NTDS and DCSync](ntds-and-dcsync.md)**: the whole domain's hashes from the DC or by replication.
- **[Cracking](cracking.md)**: turning recovered hashes into passwords offline.
- **[User and group enumeration](user-and-group-enumeration.md)**: building the account list to target.
- **[Password policy](password-policy.md)**: reading the policy that bounds safe spraying.
- **[Password spraying](password-spraying.md)**: guessing valid credentials without lockout.

## References

- The Hacker Recipes: credential dumping and cracking
- Microsoft: credential storage and protection
