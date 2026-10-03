---
title: "Entra hybrid: pivoting between on-premises AD and the cloud"
description: "Abusing Microsoft Entra Connect and hybrid identity to move between on-premises Active Directory and the Entra ID tenant: the sync account's DCSync rights, the Seamless SSO computer account for cloud impersonation, and primary refresh token theft."
keywords:
  - Entra Connect
  - Azure AD Connect
  - MSOL account
  - AZUREADSSOACC
  - primary refresh token
---

# Entra hybrid

Most domains are no longer islands: **Entra Connect** (formerly Azure AD Connect) synchronises on-premises AD into an Entra ID (Azure AD) tenant, and that bridge is a two-way attack path. The **Connect server** is a Tier-0 asset that is routinely protected like an ordinary member server, yet it holds the keys to both directories. Owning it, or the accounts it creates, pivots from on-premises Domain Admin to cloud Global Administrator and back.

## The sync account holds DCSync

Entra Connect creates an on-premises connector account (named `MSOL_` or `AAD_`). **When password hash synchronization is enabled**, it holds **`Replicate Directory Changes`** and **`Replicate Directory Changes All`** on the domain, which is [DCSync](../authentication/credentials/ntds-and-dcsync.md). From local admin on the Connect server, recover that account's cleartext credentials from the sync configuration, then DCSync the whole domain (including krbtgt):

```powershell
# AADInternals: extract the sync credentials from the Connect server
Get-AADIntSyncCredentials
# -> on-prem connector (MSOL_) account + cloud sync account creds; use the MSOL_ cred for DCSync
```

A basic-synchronization, pass-through-authentication, or federated install without hash sync may not grant the connector `Replicate Directory Changes All`, so confirm the account's rights before relying on the DCSync path.

## Seamless SSO: the AZUREADSSOACC$ account

If Seamless SSO is enabled, a computer account **`AZUREADSSOACC$`** exists in on-premises AD. Its key signs Kerberos tickets for the cloud SSO service, so its NT hash (via DCSync) lets you forge a **silver ticket** impersonating **any synced user to the cloud**:

```powershell
# With the AZUREADSSOACC$ hash, forge a ticket for the Azure AD SSO SPN and ride it into the tenant
# (AADInternals Open-AADIntOffice365Portal / ticket forging against the aadg.windows.net.nsatc.net SPN)
```

## Primary refresh tokens

On a joined endpoint, the **primary refresh token (PRT)** is the device's cloud SSO credential. Stealing it (with the matching session key) gives cloud access as that user without their password or MFA, a direct on-prem-to-cloud pivot from a workstation.

## AD FS: Golden SAML

Where the tenant federates through on-premises **AD FS**, the AD FS **token-signing certificate** signs the SAML tokens the cloud trusts. Steal that certificate's private key and you **forge SAML tokens for any user**, with any claims, bypassing the password and MFA and generating no authentication event at the identity provider. This is **Golden SAML**, the technique used in the 2020 SolarWinds intrusions.

The signing certificate is stored encrypted, protected by the **DKM** (Distributed Key Manager) master key held in Active Directory, so forging it needs admin on the AD FS server (or the DKM key plus the encrypted configuration from AD):

```powershell
# Run ON the AD FS server (admin): export the token-signing certificate to a PFX, then forge
Export-AADIntADFSSigningCertificate -FileName signing.pfx
New-AADIntSAMLToken -ImmutableID <user-immutableid> -PfxFileName signing.pfx -Issuer "http://adfs.example.local/adfs/services/trust"
# remotely, export the configuration + DKM key first (Export-AADIntADFSConfiguration / -ADFSEncryptionKey);
# ADFSDump + ADFSpoof are the equivalent standalone toolchain
```

## Exploitation notes

- The Connect server is effectively **both a domain controller and a tenant admin** in reach, so compromising it is the shortest hybrid takeover; treat it as Tier-0 when scoping.
- The `MSOL_` DCSync path means the Connect server is an alternative route to the whole domain even when the DCs themselves are hard to reach.
- In **pass-through authentication** tenants, the PTA agent on the Connect server can be backdoored to intercept or validate any cloud logon, a durable authentication backdoor.
- `AADInternals` is the established toolkit once you hold local admin on the sync server.

## Tools

- **AADInternals** (Nestori Syynimaa): sync-credential extraction, token forging, PTA and SSO abuse.
- **Impacket / mimikatz**: DCSync with the recovered `MSOL_` or `AZUREADSSOACC$` material.

## References

- [AADInternals (o365blog)](https://github.com/Gerenios/AADInternals)
- [Cloud-Architekt: Entra Connect sync service account attack and defense](https://github.com/Cloud-Architekt/AzureAD-Attack-Defense/blob/main/AADCSyncServiceAccount.md)
- [Reversec: Entra Connect exploitation in 2025](https://labs.reversec.com/posts/2025/10/entra-connect-exploitation-in-2025-an-overview)
- [inversecos: backdooring Office 365 and Active Directory with Golden SAML](https://www.inversecos.com/2021/09/backdooring-office-365-and-active.html)
