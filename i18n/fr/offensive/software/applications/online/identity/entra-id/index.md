---
title: "Entra ID"
description: "Attaquer les tenants Microsoft Entra ID (Azure AD) : abus d'authentification et de jetons, prise de contrôle d'applications et de service principals, élévation de privilèges via les rôles d'annuaire et les groupes, enregistrement d'appareils, et accès inter-tenant et invité."
keywords:
  - Entra ID
  - Azure AD
  - tenant
  - service principal
  - PRT
---

# Entra ID

Microsoft Entra ID (anciennement Azure AD) est l'**annuaire** cloud derrière Microsoft 365 et Azure : le plan d'identité à l'échelle du tenant des utilisateurs, groupes, applications, service principals et appareils, atteint via Microsoft Graph et les endpoints de connexion. Il s'attaque comme un annuaire, pas comme une plateforme de ressources cloud, ce qui explique pourquoi il se trouve ici à côté d'Active Directory plutôt que sous Cloud.

Les attaques Entra se déroulent de l'extérieur vers l'intérieur : énumérer le tenant et ses utilisateurs, obtenir un premier jeton via le spraying ou le phishing, puis s'élever à travers les applications sur-permissionnées, les rôles d'annuaire, les groupes et les appareils jusqu'à détenir Global Administrator ou une application qui le détient. La monnaie d'échange tout au long est le **jeton** (d'accès, de rafraîchissement, et le primary refresh token), pas un mot de passe.

## Frontière avec le plan de ressources Azure

Cette zone est le plan **tenant et annuaire**. Le plan **ressources** Azure (abonnements, VM, stockage, Azure RBAC, identités managées) s'attaque séparément et se trouve sous [Cloud > Azure](../../../online/cloud/azure/index.md). Les deux se rejoignent à la couture `elevateAccess`, où un Global Administrator s'accorde User Access Administrator sur tous les abonnements ; ce franchissement est noté là où il convient. Le pont on-premises (Entra Connect, AD FS, PRT, pass-through auth) se trouve avec l'[identité hybride](../../../server/directory/active-directory/trusts/entra-hybrid.md).

## Énumération

L'énumération se fond dans chaque surface, mais la reconnaissance à l'échelle du tenant est exécutée en premier : découverte non authentifiée du tenant et des utilisateurs avec **AADInternals** et les endpoints `GetCredentialType` et de configuration OpenID, puis énumération authentifiée des utilisateurs, groupes, applications, service principals et rôles avec **ROADtools** (`roadrecon`) et **AzureHound** pour le graphe d'attaque.

## Sections

- **[Authentification](authentication/index.md)** : énumération des utilisateurs et du tenant, password spraying, phishing par device-code et de jetons, primary refresh tokens, et contournement de l'accès conditionnel et de la MFA.
- **[Applications](applications/index.md)** : phishing de consentement, abus des permissions Microsoft Graph, identifiants ajoutés et fédérés, et prise de contrôle de service principals et de propriété.
- **[Rôles](roles/index.md)** : rôles d'annuaire privilégiés, activation PIM, et cadrage par unité administrative.
- **[Groupes](groups/index.md)** : injection de règle d'appartenance dynamique, propriété de groupe, et groupes role-assignable.
- **[Appareils](devices/index.md)** : enregistrement et jointure d'appareils pirates, jetons d'enregistrement en masse, et falsification de jointure hybride pour un PRT.
- **[Tenant](tenant/index.md)** : accès invité B2B et politiques d'accès inter-tenant.

## Références

- [HackTricks Cloud : Azure and Entra](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [AADInternals (Dr. Nestori Syynimaa)](https://aadinternals.com/aadinternals/)
- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
- [SpecterOps : AzureHound and BARK](https://github.com/BloodHoundAD/AzureHound)
