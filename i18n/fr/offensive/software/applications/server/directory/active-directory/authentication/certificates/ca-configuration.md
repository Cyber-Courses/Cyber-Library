---
order: 4
title: "Configuration de la CA : ESC6 et ESC16"
description: "Abuser des drapeaux de configuration d'une CA d'entreprise : EDITF_ATTRIBUTESUBJECTALTNAME2 qui permet à toute requête de spécifier son propre SAN (ESC6), et une CA qui désactive l'extension de sécurité SID pour tous les certificats émis (ESC16)."
keywords:
  - ESC6
  - ESC16
  - EDITF_ATTRIBUTESUBJECTALTNAME2
  - security extension
  - certificate authority
---

# Configuration de la CA

Certaines escalades AD CS proviennent des **paramètres propres à l'autorité de certification**, qui s'appliquent à chaque certificat qu'elle émet, quel que soit le modèle. Un seul drapeau de CA incorrect transforme des modèles par ailleurs sûrs en chemins d'escalade sur l'ensemble de l'environnement.

## ESC6 : le drapeau d'attribut SAN

Lorsqu'une CA a `EDITF_ATTRIBUTESUBJECTALTNAME2` activé, elle honore un **subjectAltName fourni dans la requête** pour *n'importe quel* modèle, pas seulement ceux configurés pour un sujet fourni par l'inscrit. Cela reproduit [ESC1](vulnerable-templates.md) partout : inscrivez-vous pour n'importe quel modèle d'authentification client et attachez un SAN nommant un administrateur de domaine :

```bash
# With the CA flag set, specify the SAN on a normally safe template
certipy req -u user@example.local -p pass -ca <ca> -template User \
  -upn administrator@example.local
```

Le drapeau est lu lors de l'[énumération](enumeration.md) et peut être **défini** par un attaquant détenant les droits ManageCA d'[ESC7](access-control.md), ESC6 et ESC7 se chaînent donc.

Notez que l'extension de sécurité SID complique un ESC6 naïf sur les domaines corrigés : lorsque la liaison forte de certificat est appliquée, un SAN seul peut ne pas permettre l'usurpation, et ESC6 se combine alors avec une [faiblesse de mappage](certificate-mapping.md).

## ESC16 : extension de sécurité désactivée sur la CA

ESC16 est la version à l'échelle de la CA d'[ESC9](certificate-mapping.md) : la CA est configurée pour **omettre l'extension de sécurité SID** de chaque certificat qu'elle émet (l'OID de l'extension figure dans la liste des extensions désactivées de la CA). Comme pour ESC9, cela ne permet l'usurpation que lorsque le DC autorise encore le mappage faible (`StrongCertificateBindingEnforcement` à `0` ou `1`) ; sous application complète (`2`), un certificat sans la liaison SID est rejeté. Sur un DC à mappage faible, l'usurpation par échange d'UPN fonctionne alors contre toute la CA plutôt que contre un seul modèle :

```bash
# Enrol after setting a controlled account's UPN to the victim; the issued cert
# lacks the SID binding, so it maps to the victim by UPN
certipy account update -u user@example.local -p pass -user puppet -upn administrator@example.local
certipy req -u puppet@example.local -p pass2 -ca <ca> -template User
```

## Notes d'exploitation

- ESC6 est un chemin à un seul drapeau vers l'administrateur de domaine sur tous les modèles d'authentification client, c'est pourquoi il est de grande valeur lorsque `certipy find` le signale.
- ESC6 défini via ESC7 nécessite un **redémarrage du service CA** pour prendre effet, ce qui est bruyant et peut requérir l'hôte de la CA ; mettez cela en balance avec des chemins de modèle plus discrets.
- ESC16 transforme chaque certificat en candidat ESC9, il est donc aussi grave qu'une défaillance de mappage à l'échelle du domaine ; associez-le à tout droit d'inscription et à un UPN accessible en écriture.

## Outils

- **Certipy** (`req`, `ca`, `account update`) : s'inscrire avec un SAN choisi, basculer les drapeaux de CA (avec ESC7) et échanger les UPN.
- **certutil** (`-setreg`) : lecture/définition native des drapeaux de politique de la CA.

## Références

- [SpecterOps: Certified Pre-Owned (ESC6)](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
- [Certipy (ly4k)](https://github.com/ly4k/Certipy)
- [Microsoft: Active Directory Certificate Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/)
