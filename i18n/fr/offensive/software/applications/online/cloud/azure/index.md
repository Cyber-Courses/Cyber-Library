---
title: "Azure"
description: "Attaquer le plan de ressources et de gestion de Microsoft Azure : élévation de privilèges Azure RBAC et identité managée, vol de credentials depuis IMDS et Key Vault, et abus des services de compute, de stockage, serverless, de données, réseau, de journalisation et de messagerie d'un abonnement."
keywords:
  - Azure
  - ARM
  - Azure RBAC
  - identité managée
  - abonnement
---

# Azure

Azure est piloté via l'API Azure Resource Manager (**ARM**), autorisée par les attributions de rôle **Azure RBAC** et atteinte avec un token utilisateur, un secret de principal de service ou l'**identité managée** d'une ressource. Presque chaque attaque est une question de RBAC : quel scope mon principal détient-il, quel rôle lui permet d'escalader, et quelle identité managée de ressource peut-il emprunter. Les surfaces ci-dessous suivent cette forme, l'**élévation de privilèges** résidant au sein de l'[identité](identity/index.md) et le vol de credentials au sein des [credentials](credentials/index.md).

Cette section concerne **uniquement le plan de ressources et de gestion**. Microsoft Entra ID (l'annuaire du tenant, anciennement Azure AD) s'attaque comme un annuaire et se trouve sous [Entra ID](../../identity/entra-id/index.md) à côté d'Active Directory ; les attaques sur l'identité des utilisateurs et des applications (device code, PRT, consentement, rôles Entra) y sont, pas ici.

## Énumération

L'énumération se fond dans chaque surface, mais l'inventaire à l'échelle de l'abonnement est exécuté en premier : `az account list` et `az resource list` pour le resource graph, les requêtes **Azure Resource Graph** pour le passage à l'échelle, les attributions de rôle avec `az role assignment list --all`, et les identités managées avec **ROADtools**, **Stormspotter** ou **AzureHound** (BARK) pour le graphe RBAC et des ressources. Tout cela alimente chaque surface ci-dessous.

## Surfaces

- **[Identité](identity/index.md)** : écritures de rôle et de rôle personnalisé Azure RBAC, elevate-access, attribution d'identité managée et activation PIM.
- **[Credentials](credentials/index.md)** : tokens d'identité managée depuis IMDS, Key Vault, clés de stockage, assets Automation et paramètres d'application.
- **[Compute](compute/index.md)** : run command et extensions de VM, AKS, Container Registry, Container Instances et scale sets.
- **[Stockage](storage/index.md)** : énumération et accès aux blobs, tokens SAS, snapshots de disque et partages de fichiers.
- **[Serverless](serverless/index.md)** : Functions, Logic Apps, Automation Accounts, App Service et Deployment Scripts.
- **[Données](data/index.md)** : SQL Database, Cosmos DB, Data Factory, Synapse et tables de stockage.
- **[Réseau](networking/index.md)** : network security groups et VNet, private endpoints, DNS takeover, Front Door et Bastion.
- **[Journalisation et détection](logging-and-detection/index.md)** : altération de l'Activity Log, d'Azure Monitor, de Defender for Cloud et de Sentinel.
- **[Messagerie](messaging/index.md)** : Service Bus, Event Hubs, Event Grid et Storage Queues.

## Références

- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [SpecterOps : BARK et AzureHound](https://github.com/BloodHoundAD/BARK)
- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
