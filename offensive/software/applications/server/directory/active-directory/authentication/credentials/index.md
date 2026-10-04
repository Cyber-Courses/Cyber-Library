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
- **The directory**: passwords in attributes, LAPS and gMSA managed passwords (covered under [credentials in attributes](../../../ldap/protocol/credentials-in-attributes.md)).

## Pages

- **[Token impersonation](token-impersonation.md)**: SeImpersonate to SYSTEM from a service-account foothold (the potato family).
- **[LSASS dumping](lsass-dumping.md)**: extracting live credentials and tickets from LSASS memory.
- **[SAM and LSA secrets](sam-and-lsa-secrets.md)**: local hashes and cached machine and service secrets.
- **[DPAPI secrets](dpapi-secrets.md)**: browser, vault, and application credentials.
- **[NTDS and DCSync](ntds-and-dcsync.md)**: the whole domain's hashes from the DC or by replication.
- **[ZeroLogon](zerologon.md)**: resetting the DC machine account over Netlogon for unauthenticated domain takeover.
- **[RODC](rodc.md)**: cached-credential extraction and the scoped golden ticket from a Read-Only Domain Controller.
- **[Cracking](cracking.md)**: turning recovered hashes into passwords offline.
- **[Host and domain discovery](host-and-domain-discovery.md)**: finding domain controllers, naming contexts, and the lay of the domain.
- **[LDAP enumeration](ldap-enumeration.md)**: querying the directory for objects, attributes, and abusable flags.
- **[User and group enumeration](user-and-group-enumeration.md)**: building the account list to target.
- **[Session enumeration](session-enumeration.md)**: locating logged-on users to target for credential theft and lateral movement.
- **[Null sessions](null-sessions.md)**: anonymous SMB/LDAP enumeration and RID cycling, before any credential.
- **[Password policy](password-policy.md)**: reading the policy that bounds safe spraying.
- **[Password spraying](password-spraying.md)**: guessing valid credentials without lockout.
- **[Pre-created computers](pre-created-computers.md)**: taking over pre-staged computer objects with their default name-as-password.

Once you hold the directory's secrets, the same access supports **domain persistence**:

- **[DCShadow](dcshadow.md)**: push arbitrary changes through a rogue domain controller.
- **[DSRM](dsrm.md)**: the domain controller's local backdoor account.
- **[Skeleton Key](skeleton-key.md)**: a master password patched into LSASS.
- **[Custom SSP](custom-ssp.md)**: logging cleartext credentials on the host.

## References

- [adsecurity: Mimikatz and command reference](https://adsecurity.org/?page_id=1821)
- [Impacket (fortra): secretsdump and the credential toolkit](https://github.com/fortra/impacket)
- [NetExec (Pennyw0rth)](https://github.com/Pennyw0rth/NetExec)
- [The Hacker Recipes: Active Directory movement](https://www.thehacker.recipes/ad/movement/)
