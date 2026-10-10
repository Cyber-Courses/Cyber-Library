---
title: "Entra hybride : pivoter entre l'AD on-premises et le cloud"
order: 5
description: "Abuser de Microsoft Entra Connect et de l'identité hybride pour se déplacer entre l'Active Directory on-premises et le tenant Entra ID : les droits DCSync du compte de synchronisation, le compte d'ordinateur Seamless SSO pour l'usurpation côté cloud, et le vol de primary refresh token."
keywords:
  - Entra Connect
  - Azure AD Connect
  - MSOL account
  - AZUREADSSOACC
  - primary refresh token
---

# Entra hybride

La plupart des domaines ne sont plus des îles : **Entra Connect** (anciennement Azure AD Connect) synchronise l'AD on-premises vers un tenant Entra ID (Azure AD), et ce pont est un chemin d'attaque bidirectionnel. Le **serveur Connect** est un actif Tier-0 qui est régulièrement protégé comme un simple serveur membre, alors qu'il détient les clés des deux annuaires. En prendre le contrôle, ou celui des comptes qu'il crée, permet de pivoter du Domain Admin on-premises au Global Administrator cloud et inversement.

## Le compte de synchronisation détient DCSync

Entra Connect crée un compte connecteur on-premises (nommé `MSOL_` ou `AAD_`). **Lorsque la synchronisation des empreintes de mot de passe est activée**, il détient **`Replicate Directory Changes`** et **`Replicate Directory Changes All`** sur le domaine, ce qui correspond à [DCSync](../authentication/credentials/ntds-and-dcsync.md). Depuis un compte administrateur local sur le serveur Connect, récupérez les identifiants en clair de ce compte depuis la configuration de synchronisation, puis faites un DCSync de tout le domaine (y compris krbtgt) :

```powershell
# AADInternals: extract the sync credentials from the Connect server
Get-AADIntSyncCredentials
# -> on-prem connector (MSOL_) account + cloud sync account creds; use the MSOL_ cred for DCSync
```

Une installation en synchronisation basique, en authentification pass-through ou fédérée sans synchronisation d'empreintes peut ne pas accorder au connecteur `Replicate Directory Changes All`, confirmez donc les droits du compte avant de vous fier au chemin DCSync.

## Seamless SSO : le compte AZUREADSSOACC$

Si Seamless SSO est activé, un compte d'ordinateur **`AZUREADSSOACC$`** existe dans l'AD on-premises. Sa clé signe les tickets Kerberos du service SSO cloud, de sorte que son empreinte NT (via DCSync) vous permet de forger un **silver ticket** usurpant **n'importe quel utilisateur synchronisé vers le cloud** :

```powershell
# With the AZUREADSSOACC$ hash, forge a ticket for the Azure AD SSO SPN and ride it into the tenant
# (AADInternals Open-AADIntOffice365Portal / ticket forging against the aadg.windows.net.nsatc.net SPN)
```

## Primary refresh tokens

Sur un terminal joint, le **primary refresh token (PRT)** est l'identifiant SSO cloud de l'appareil. Le voler (avec la clé de session correspondante) donne un accès cloud en tant que cet utilisateur sans son mot de passe ni MFA, un pivot direct de l'on-premises vers le cloud depuis une station de travail.

## AD FS : Golden SAML

Lorsque le tenant est fédéré via un **AD FS** on-premises, le **certificat de signature de jetons** d'AD FS signe les jetons SAML auxquels le cloud fait confiance. Volez la clé privée de ce certificat et vous pouvez **forger des jetons SAML pour n'importe quel utilisateur**, avec n'importe quelles revendications, en contournant le mot de passe et le MFA et sans générer aucun événement d'authentification chez le fournisseur d'identité. C'est **Golden SAML**, la technique employée dans les intrusions SolarWinds de 2020.

Le certificat de signature est stocké chiffré, protégé par la clé maîtresse **DKM** (Distributed Key Manager) détenue dans Active Directory, de sorte que le forger nécessite un compte administrateur sur le serveur AD FS (ou la clé DKM plus la configuration chiffrée issue de l'AD) :

```powershell
# Run ON the AD FS server (admin): export the token-signing certificate to a PFX, then forge
Export-AADIntADFSSigningCertificate -FileName signing.pfx
New-AADIntSAMLToken -ImmutableID <user-immutableid> -PfxFileName signing.pfx -Issuer "http://adfs.example.local/adfs/services/trust"
# remotely, export the configuration + DKM key first (Export-AADIntADFSConfiguration / -ADFSEncryptionKey);
# ADFSDump + ADFSpoof are the equivalent standalone toolchain
```

## Notes d'exploitation

- Le serveur Connect est de fait **à la fois un contrôleur de domaine et un administrateur de tenant** par sa portée, le compromettre est donc la prise de contrôle hybride la plus courte ; traitez-le comme un Tier-0 lors du cadrage.
- Le chemin DCSync via `MSOL_` signifie que le serveur Connect est une voie alternative vers tout le domaine, même lorsque les DC eux-mêmes sont difficiles à atteindre.
- Dans les tenants en **authentification pass-through**, l'agent PTA sur le serveur Connect peut être piégé pour intercepter ou valider n'importe quelle connexion cloud, une porte dérobée d'authentification durable.
- `AADInternals` est la boîte à outils de référence une fois que vous disposez d'un compte administrateur local sur le serveur de synchronisation.

## Outils

- **AADInternals** (Nestori Syynimaa) : extraction des identifiants de synchronisation, forge de jetons, abus de PTA et de SSO.
- **Impacket / mimikatz** : DCSync avec le matériel `MSOL_` ou `AZUREADSSOACC$` récupéré.

## Références

- [AADInternals (o365blog)](https://github.com/Gerenios/AADInternals)
- [Cloud-Architekt: Entra Connect sync service account attack and defense](https://github.com/Cloud-Architekt/AzureAD-Attack-Defense/blob/main/AADCSyncServiceAccount.md)
- [Reversec: Entra Connect exploitation in 2025](https://labs.reversec.com/posts/2025/10/entra-connect-exploitation-in-2025-an-overview)
- [inversecos: backdooring Office 365 and Active Directory with Golden SAML](https://www.inversecos.com/2021/09/backdooring-office-365-and-active.html)
