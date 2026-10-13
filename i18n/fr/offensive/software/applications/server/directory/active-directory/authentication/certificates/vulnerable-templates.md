---
order: 1
title: "Modèles vulnérables : ESC1, ESC2, ESC3, ESC13, ESC15"
description: "Abuser de modèles de certificat AD CS mal configurés pour s'inscrire pour un certificat qui s'authentifie en tant que compte privilégié : sujet fourni par l'inscrit (ESC1), EKU trop larges (ESC2), agents d'inscription (ESC3), liens de groupe par OID (ESC13), et injection de politique d'application (ESC15)."
keywords:
  - ESC1
  - ESC2
  - ESC3
  - ESC13
  - ESC15
---

# Modèles vulnérables

Un modèle de certificat définit à quoi sert un certificat et qui peut le demander. Lorsqu'un modèle combine un **droit d'inscription peu privilégié** avec des paramètres qui permettent au demandeur de choisir *pour qui est le certificat* ou *ce qu'il peut faire*, tout utilisateur de ce type peut s'inscrire pour un certificat qui s'authentifie en tant que compte privilégié. Ce sont les escalades AD CS les plus directes.

## ESC1 : sujet fourni par l'inscrit

Le cas classique. Un modèle qui (1) accorde l'inscription aux utilisateurs peu privilégiés, (2) possède un **EKU d'authentification client** (Client Authentication, Smart Card Logon, PKINIT Client Authentication, ou Any Purpose), et (3) laisse l'**inscrit fournir le sujet** (`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`). Deux verrous d'émission doivent aussi être ouverts : **approbation du gestionnaire désactivée** (sinon la requête est retenue en attente d'un gestionnaire de certificats) et **aucune signature autorisée requise** (sinon la requête nécessite un certificat d'agent d'inscription co-signataire). Avec tout cela, vous demandez un certificat et spécifiez un **SAN** nommant un administrateur de domaine :

```bash
certipy req -u user@example.local -p pass -ca <ca-name> -template <vuln-template> \
  -upn administrator@example.local
# -> administrator.pfx, then authenticate as the DA
certipy auth -pfx administrator.pfx -dc-ip <dc>
```

## ESC2 et ESC3 : finalités trop larges et agents d'inscription

- **ESC2** : un modèle doté de l'EKU **Any Purpose** (ou sans aucun EKU, un modèle SubCA). Le certificat émis peut servir à l'authentification client même si rien n'indique « auth client », de sorte qu'un sujet fourni par l'inscrit ou une faiblesse de mappage le transforme en usurpation.
- **ESC3** : un modèle accordant l'EKU **Certificate Request Agent**. Vous vous inscrivez pour un certificat d'agent d'inscription, puis l'utilisez pour vous inscrire **au nom** d'un autre utilisateur contre un modèle qui accepte les agents d'inscription :

```bash
certipy req -u user@example.local -p pass -ca <ca> -template <agent-template>
certipy req -u user@example.local -p pass -ca <ca> -template User \
  -on-behalf-of 'EXAMPLE\administrator' -pfx agent.pfx
```

## ESC13 : politique d'émission liée à un groupe

Un modèle avec une **politique d'émission** dont l'OID est lié à un groupe AD (`msDS-OIDToGroupLink`). Un certificat émis à partir de ce modèle accorde à son détenteur les privilèges du groupe lié, de sorte que s'inscrire (même pour soi-même) produit un jeton avec cette appartenance de groupe, sans aucune manipulation de sujet.

## ESC15 : injection de politique d'application (EKUwu)

Sur les modèles de schéma **version 1** servis par une **CA encore vulnérable à la faille d'injection de politique d'application** (les CA corrigées ignorent les politiques injectées), le demandeur peut injecter des **politiques d'application** dans le CSR que la CA inclut dans le certificat, même lorsque l'EKU du modèle ne permettrait pas l'authentification client. Ajouter une politique d'application Client Authentication produit un certificat capable d'authentification à partir d'un modèle qui semblait sûr. Nommer un autre principal nécessite toujours que le modèle laisse l'inscrit fournir le sujet ; sinon, la politique injectée ne fait qu'améliorer un certificat pour vous-même :

```bash
certipy req -u user@example.local -p pass -ca <ca> -template <v1-template> \
  -application-policies 'Client Authentication' -upn administrator@example.local
```

## Notes d'exploitation

- ESC1 et ESC15 aboutissent tous deux à un certificat nommant un administrateur de domaine ; enchaînez avec `certipy auth` (PKINIT) et, si utile, [UnPAC-the-hash](../kerberos/unpac-the-hash.md) pour récupérer le hash NT de la cible.
- ESC3 est précieux lorsqu'aucun modèle à sujet fourni par l'inscrit n'existe : le chemin de l'agent d'inscription atteint les utilisateurs privilégiés via un modèle normalement bénin.
- L'extension de sécurité SID (voir [mappage de certificat](certificate-mapping.md)) peut bloquer l'usurpation naïve par SAN sur les domaines corrigés, c'est pourquoi ESC1 est souvent associé à ESC9/ESC16 ou déplacé vers ESC15.

## Outils

- **Certipy** (`req`, `auth`, `-on-behalf-of`, `-application-policies`) : s'inscrire et s'authentifier pour toutes ces techniques.
- **Certify / ForgeCert** : inscription et forge côté Windows.

## Références

- [SpecterOps: Certified Pre-Owned (ESC1-ESC3)](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
- [TrustedSec: EKUwu, not just another AD CS ESC (ESC15)](https://trustedsec.com/blog/ekuwu-not-just-another-ad-cs-esc)
- [Certipy (ly4k)](https://github.com/ly4k/Certipy)
