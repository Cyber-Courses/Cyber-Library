---
order: 3
title: "Contrôle d'accès : ESC4, ESC5, ESC7"
description: "Abuser des accès en écriture sur les objets AD CS : reconfigurer un modèle via son ACL (ESC4), contrôler les objets CA ou PKI et l'hôte de la CA (ESC5), et abuser des droits de rôle CA ManageCA et ManageCertificates (ESC7)."
keywords:
  - ESC4
  - ESC5
  - ESC7
  - ManageCA
  - ACL de modèle de certificat
---

# Contrôle d'accès

Plusieurs escalades AD CS ne reposent pas sur une mauvaise configuration permanente mais sur un **accès en écriture** aux objets qui contrôlent la certification. Une permission sur un modèle, un objet CA ou un rôle CA vous permet de *créer* vous-même la condition vulnérable, puis de vous inscrire. Ce sont les chemins que BloodHound fait ressortir sous forme d'objets AD CS accessibles en écriture.

## ESC4 : modèle accessible en écriture

Si vous disposez d'un accès en écriture (`GenericWrite`, `GenericAll`, `WriteDacl`, `WriteOwner`) sur un objet **modèle de certificat**, vous pouvez réécrire ses paramètres pour le rendre vulnérable à [ESC1](vulnerable-templates.md) : activer le sujet fourni par l'inscrit, ajouter un EKU d'authentification client, et vous accorder l'inscription. Inscrivez-vous ensuite en tant qu'administrateur de domaine et, idéalement, rétablissez le modèle :

```bash
# Certipy can reconfigure a template to be vulnerable, enrol, and restore it
certipy template -u user@example.local -p pass -template <template> -write-default-configuration
# ... enrol with -upn administrator@example.local, then restore the saved config
```

## ESC5 : contrôle des objets CA ou de l'hôte de la CA

ESC5 est la catégorie large du contrôle des objets dont dépend la PKI, en dehors des modèles eux-mêmes :

- L'**objet ordinateur de la CA** ou l'hôte du serveur CA (être administrateur local sur la CA permet de lire sa clé privée et d'émettre des certificats arbitraires).
- Les **objets AD de la CA** (les objets `pKIEnrollmentService` / `certificationAuthority`, les conteneurs PKI).
- L'objet **NTAuthCertificates**, qui liste les CA de confiance pour l'authentification ; l'écrire permet d'ajouter une CA malveillante à la confiance de la forêt.

Le contrôle de l'un de ces éléments équivaut au contrôle de l'émission des certificats, il est donc traité comme une primitive d'escalade de domaine.

## ESC7 : droits de rôle CA

La CA elle-même possède deux rôles puissants dont les droits peuvent être délégués :

- **ManageCA** (administrateur de CA) : peut modifier la configuration de la CA, y compris définir le drapeau `EDITF_ATTRIBUTESUBJECTALTNAME2` qui active [ESC6](ca-configuration.md), et peut activer les approbations du gestionnaire de certificats.
- **ManageCertificates** (gestionnaire de certificats) : peut **approuver les requêtes en attente**, de sorte qu'une requête qui serait retenue pour approbation peut être approuvée par le même attaquant.

Détenir les deux (ou ManageCA pour vous accorder ManageCertificates) vous permet de soumettre une requête pour une identité privilégiée et de l'approuver :

```bash
# Certipy: use ManageCA/ManageCertificates to issue despite restrictions
certipy ca -u user@example.local -p pass -ca <ca> -add-officer user       # grant cert-manager
certipy req ... -template <template>                                        # submit
certipy ca -u user@example.local -p pass -ca <ca> -issue-request <id>       # approve
```

## Notes d'exploitation

- ESC4 est le plus propre : un modèle accessible en écriture devient ESC1, cherchez donc les accès en écriture sur les modèles lors de l'[énumération des ACL](../../dacl/acl-enumeration.md).
- Le chemin ManageCA d'ESC7 peut aussi **activer le point de terminaison d'inscription web**, ouvrant la porte au [relais ESC8](relay-to-adcs.md).
- Rétablissez toujours un modèle ou un paramètre de CA reconfiguré ensuite pour limiter les perturbations et garder le changement discret.

## Outils

- **Certipy** (`template`, `ca`) : reconfigurer les modèles, gérer les rôles CA, approuver les requêtes.
- **PSPKI / Certify** : gestion de la CA et des modèles côté Windows.

## Références

- [SpecterOps: Certified Pre-Owned (ESC4, ESC5, ESC7)](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
- [Certipy (ly4k)](https://github.com/ly4k/Certipy)
- [SpecterOps: Certify 2.0](https://specterops.io/blog/2025/08/11/certify-2-0/)
