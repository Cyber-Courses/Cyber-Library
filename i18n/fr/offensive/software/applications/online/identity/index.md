---
title: "Identité : attaquer les fournisseurs d'identité hébergés par des éditeurs"
order: 2
description: "Attaquer les fournisseurs d'identité et les annuaires hébergés dans le cloud : des tenants atteints via l'authentification et l'abus de jetons, l'abus des applications et du consentement OAuth, l'escalade de privilèges via les rôles d'annuaire et les groupes, et l'accès inter-tenant. Il n'y a pas de serveur d'annuaire à atteindre, seulement la surface de connexion et les API du fournisseur."
keywords:
  - identité cloud
  - fournisseur d'identité
  - Entra ID
  - Azure AD
  - tenant
  - OAuth
---

# Identité

Un fournisseur d'identité hébergé par un éditeur est un annuaire que vous ne pouvez pas atteindre en tant que serveur : il n'y a pas de contrôleur de domaine à toucher, seulement les points de terminaison de connexion, le service de jetons et les API de gestion du fournisseur. Le modèle d'attaque est donc centré sur l'identité et sur les jetons : énumérer le tenant, obtenir un principal via une identification pulvérisée (spray) ou un jeton hameçonné, puis escalader via le consentement d'application, les rôles d'annuaire et l'appartenance aux groupes, et franchir les frontières des tenants via la confiance invité et de fédération. Cela se situe sous Online parce que l'annuaire s'exécute sur l'infrastructure de l'éditeur, mais les techniques reflètent les attaques d'annuaire sous Server plutôt que le travail sur le plan des ressources sous Cloud.

## Triage

```bash
# Is a target tenant Microsoft Entra ID, and does it federate?
curl -s "https://login.microsoftonline.com/<domain>/.well-known/openid-configuration" | python3 -m json.tool | head
curl -s "https://login.microsoftonline.com/getuserrealm.srf?login=user@<domain>&xml=1"   # Managed vs Federated
```

Un tenant Microsoft route vers [Entra ID](entra-id/index.md). L'annuaire sur site depuis lequel il peut se synchroniser (Active Directory) est une cible auto-hébergée sous [Directory](../../server/directory/index.md), et le serveur de synchronisation reliant les deux est traité dans les pages sur l'identité hybride AD.

## Sous-sujets

- **[Entra ID](entra-id/index.md)** : les tenants Microsoft Entra ID (Azure AD), de l'authentification et l'abus de jetons jusqu'aux attaques sur les applications, les rôles, les groupes, les appareils et l'inter-tenant.

## Références

- [Documentation Microsoft Entra](https://learn.microsoft.com/en-us/entra/)
- [MITRE ATT&CK : matrice Cloud](https://attack.mitre.org/matrices/enterprise/cloud/)
