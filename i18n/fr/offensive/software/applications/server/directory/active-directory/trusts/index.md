---
title: "Approbations : se déplacer à travers les domaines et forêts Active Directory"
order: 4
description: "Les relations d'approbation Active Directory comme chemin entre domaines et forêts : clés d'approbation et tickets inter-domaines, SID history à l'intérieur d'une forêt, et les chemins plus étroits à travers les approbations de forêt bornés par le filtrage de SID."
keywords:
  - active directory trusts
  - forest trust
  - SID history
  - SID filtering
  - inter-realm TGT
---

# Approbations

Une approbation permet aux principaux d'un domaine de s'authentifier auprès de ressources d'un autre. Les approbations sont ce qui fait d'une **forêt** une frontière de sécurité unique et ce qui relie des forêts distinctes. Pour un attaquant, une approbation est un pont : la compromission d'un domaine s'étend fréquemment aux domaines qui lui font confiance, et les règles censées contenir cette propagation (filtrage de SID, authentification sélective) sont souvent plus faibles qu'on ne le suppose, surtout **à l'intérieur** d'une forêt.

## Le paysage des approbations

- **Approbations intra-forêt** (entre domaines d'une même forêt) sont, par conception, une frontière faible : la forêt est la véritable frontière de sécurité, l'escalade inter-domaines au sein d'une forêt est donc généralement réalisable.
- **Approbations inter-forêts** sont censées être plus solides, bornées par le **filtrage de SID**, mais les mauvaises configurations, les tolérances de SID history et les comptes partagés ouvrent des chemins à travers elles.
- **Direction et transitivité de l'approbation** décident qui peut atteindre qui ; la compromission d'un domaine approuvé se propage souvent vers le domaine approbateur.

## Comment les approbations sont franchies

- **SID history** : un SID privilégié du domaine cible, placé dans les `extraSids` d'un ticket forgé, octroie l'accès à ce domaine partout où le filtrage de SID ne le retire pas ; à l'intérieur d'une forêt, il n'est généralement pas retiré.
- **Clés d'approbation** : la clé partagée d'une approbation inter-domaines forge les tickets de référence inter-domaines qui déplacent un principal à travers l'approbation.
- **Garde-fous** : les approbations inter-forêts appliquent par défaut le filtrage de SID et l'authentification sélective, de sorte que les franchir repose sur un accès accordé ou une configuration assouplie plutôt que sur l'injection de SID.

## Pages

- **[Énumération des approbations](trust-enumeration.md)** : cartographier les approbations, leur direction, leur transitivité et leur posture de filtrage.
- **[Approbations intra-forêt](intra-forest-trusts.md)** : atteindre la racine de la forêt depuis n'importe quel domaine via le SID history.
- **[Approbations inter-forêts](cross-forest-trusts.md)** : se déplacer entre forêts sous filtrage de SID et authentification sélective.
- **[Clés d'approbation et tickets inter-domaines](trust-keys.md)** : forger des tickets de référence à partir de la clé d'approbation.
- **[Entra hybride](entra-hybrid.md)** : pivoter de l'AD on-premises vers le tenant Entra ID et inversement via Entra Connect.

## Références

- The Hacker Recipes: domain and forest trusts
- Microsoft: how Active Directory trusts work and SID filtering
