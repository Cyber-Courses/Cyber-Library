---
title: "Comptes gérés : secrets de comptes stockés dans l'annuaire et abus de dMSA"
order: 5
description: "Abuser des comptes et des mots de passe qu'Active Directory gère pour vous : lire les mots de passe des group Managed Service Accounts (gMSA) et les mots de passe d'administrateur local LAPS depuis l'annuaire, et l'escalade de privilèges BadSuccessor par Managed Service Account délégué sur Windows Server 2025."
keywords:
  - gMSA
  - LAPS
  - dMSA
  - BadSuccessor
  - managed service account
---

# Comptes gérés

Active Directory peut **gérer des secrets de comptes pour vous** : les group Managed Service Accounts (gMSA) ont leurs mots de passe générés et stockés dans l'annuaire, LAPS conserve dans l'annuaire le mot de passe d'administrateur local de chaque machine, et Windows Server 2025 ajoute les delegated Managed Service Accounts (dMSA). Pratique pour les administrateurs, et un cadeau pour un attaquant : le secret vit dans un attribut, de sorte qu'un droit de lecture vous remet le mot de passe, et le mécanisme de migration dMSA peut être détourné pour usurper purement et simplement n'importe quel compte.

## Les surfaces

- **gMSA** : le mot de passe géré est dans `msDS-ManagedPassword`, lisible uniquement par les principaux listés dans `msDS-GroupMSAMembership` (`PrincipalsAllowedToRetrieveManagedPassword`). Entrez dans cet ensemble et vous calculez l'empreinte NT du compte directement depuis l'annuaire.
- **LAPS** : le mot de passe d'administrateur local par machine est dans `ms-Mcs-AdmPwd` (hérité, en clair) ou dans les attributs `msLAPS-*` (Windows LAPS, éventuellement chiffré). Un droit de lecture sur l'attribut de l'ordinateur équivaut à administrateur local sur cet hôte.
- **dMSA (BadSuccessor)** : sur un domaine Server 2025, écrire les attributs de migration d'un dMSA fait construire par le KDC son PAC à partir des SID d'un compte **remplacé**, de sorte qu'un droit de création/écriture peu privilégié devient l'usurpation de n'importe quel compte, y compris Domain Admin.

## Ce qui les relie

Les trois sont **pilotés par l'annuaire** : le secret ou la décision de confiance vit dans les attributs d'un objet, l'attaque est donc une lecture ou une écriture LDAP, pas du code sur un hôte. Cela les rend discrets, compatibles avec Linux, et dépendants uniquement de la bonne ACE, c'est pourquoi ils relèvent de l'énumération [DACL](../dacl/index.md) : les arêtes `ReadGMSAPassword` et `ReadLAPSPassword` de BloodHound pointent droit vers eux.

## Pages

- **[gMSA](gmsa.md)** : lire `msDS-ManagedPassword` pour récupérer la clé d'un compte de service.
- **[LAPS](laps.md)** : lire les mots de passe d'administrateur local depuis l'annuaire.
- **[BadSuccessor](badsuccessor.md)** : abus de la migration dMSA pour l'escalade de privilèges sur Server 2025.
- **[Golden gMSA](golden-gmsa.md)** : forger hors ligne le mot de passe de n'importe quel gMSA à partir de la clé racine KDS.

## Références

- [Akamai: BadSuccessor, abusing dMSA for privilege escalation](https://www.akamai.com/blog/security-research/abusing-dmsa-for-privilege-escalation-in-active-directory)
- [SpecterOps: ReadGMSAPassword](https://bloodhound.specterops.io/resources/edges/read-gmsa-password) and [ReadLAPSPassword](https://bloodhound.specterops.io/resources/edges/read-laps-password) edges
- [Microsoft: Delegated Managed Service Accounts overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/delegated-managed-service-accounts/delegated-managed-service-accounts-overview)
