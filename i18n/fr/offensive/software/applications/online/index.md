---
title: "Online : attaquer les infrastructures et services hébergés par des éditeurs"
order: 2
description: "Attaquer des services exploités par un tiers : plateformes de fournisseurs cloud (IaaS) et applications SaaS, atteintes via l'identité, OAuth, les API et la configuration du tenant plutôt qu'en exploitant un binaire que vous pouvez atteindre. Le modèle d'attaque est le plan de contrôle et le graphe d'identité, pas l'exploitation de l'hôte."
keywords:
  - sécurité cloud
  - sécurité SaaS
  - IAM
  - OAuth
  - configuration du tenant
  - plan de contrôle
---

# Online

Les services en ligne s'exécutent sur une infrastructure exploitée par un éditeur, il n'y a donc pas de binaire serveur à atteindre et à exploiter. Ce que vous attaquez à la place, c'est le compte : les identités qui détiennent des permissions, les identifiants et les jetons qui les représentent, l'OAuth et la fédération qui les accordent, et la configuration du tenant qui les régit. C'est le même modèle centré sur le plan de contrôle et l'identité, que le service soit une infrastructure (IaaS) ou une application finie (SaaS), ce qui explique pourquoi il se distingue des services auto-hébergés sous Server.

## Où classer une cible

```bash
# Self-hosted (Server): a service you can reach as a process and exploit its software
nmap -sV <target>                 # a banner/version you could match to a CVE => Server

# Vendor-hosted (Online): a provider or SaaS endpoint, attacked through identity and API
dig +short <target>               # resolves into a provider range (AWS/Azure/GCP) or a SaaS CNAME
curl -sI https://<target>/        # vendor edge headers, a tenant login, an OAuth/SSO redirect
```

Un binaire de service accessible que vous pourriez exploiter est une cible [Server](../server/index.md). Un compte atteint via une API de fournisseur, un consentement OAuth ou un tenant SaaS est une cible Online.

## Sous-sujets

- **[Cloud](cloud/index.md)** : les plateformes de fournisseurs cloud (AWS, Azure, GCP), attaquées via l'API du plan de contrôle et le graphe IAM.
- **[Identité](identity/index.md)** : les fournisseurs d'identité et annuaires hébergés par des éditeurs (Microsoft Entra ID), attaqués via la connexion, les jetons, le consentement OAuth, les rôles d'annuaire et l'accès inter-tenant.

## Références

- [HackTricks Cloud](https://cloud.hacktricks.wiki/en/index.html)
- [MITRE ATT&CK : matrice Cloud](https://attack.mitre.org/matrices/enterprise/cloud/)
