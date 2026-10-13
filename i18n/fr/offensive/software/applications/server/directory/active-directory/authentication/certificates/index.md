---
order: 4
title: "Certificats : abuser d'AD Certificate Services"
description: "Abuser d'Active Directory Certificate Services à des fins d'authentification et d'escalade de privilèges : s'inscrire pour des certificats qui usurpent des comptes privilégiés, abuser de la configuration de la CA et du contrôle d'accès, relayer vers l'inscription, et voler des certificats."
keywords:
  - AD CS
  - ESC1
  - certificate template
  - certificate authority
  - pass the certificate
---

# Certificats

Active Directory Certificate Services (AD CS) émet des certificats que Windows accepte comme **matériel d'authentification** : un certificat doté d'une finalité d'authentification client peut être utilisé avec PKINIT pour obtenir un TGT Kerberos, de sorte que détenir le bon certificat *revient à* détenir l'identité qu'il nomme. AD CS est largement déployé, fréquemment mal configuré et rarement surveillé, c'est pourquoi la famille de techniques « Certified Pre-Owned » en fait l'un des chemins d'escalade les plus fiables vers l'administrateur de domaine.

**Filiation.** La cartographie offensive d'AD CS est arrivée en 2021 avec Certified Pre-Owned (ESC1 à ESC8). L'extension de sécurité SID et l'application de la liaison forte de certificat ont été ajoutées en 2022 pour lier plus fermement les certificats aux comptes, et ESC9, ESC10, ESC13, ESC14, ESC15 et ESC16 sont tous issus des lacunes laissées dans ce mappage et cette application.

## Pourquoi un certificat est une identité

- Un certificat lie un **sujet** (souvent un UPN ou, depuis la mise à jour de l'extension de sécurité, un SID) à une paire de clés.
- Un contrôleur de domaine mappe ce sujet sur un compte et émet un TGT (PKINIT) ou authentifie une session TLS (Schannel).
- Donc si vous pouvez obtenir un certificat qui **nomme un compte privilégié**, ou affaiblir la manière dont le mappage est appliqué, vous vous authentifiez en tant que ce compte, sans mot de passe ni hash.

Chaque attaque AD CS est une variation sur ce thème : obtenir un certificat que vous ne devriez pas avoir (abus de modèle, de CA ou de relais), ou casser le mappage censé lier un certificat à son propriétaire légitime.

## Les classes d'abus

- **Mauvaise configuration de modèle** (ESC1, ESC2, ESC3, ESC13, ESC15) : un modèle permet à un utilisateur peu privilégié de s'inscrire pour un certificat qui s'authentifie en tant qu'un autre.
- **Mappage de certificat** (ESC9, ESC10, ESC14) : une liaison SID affaiblie ou manquante permet à un certificat de se mapper sur un compte plus privilégié que celui qu'il nomme.
- **Contrôle d'accès** (ESC4, ESC5, ESC7) : un accès en écriture sur les modèles, les objets PKI ou les rôles CA est transformé en inscription.
- **Configuration de la CA** (ESC6, ESC16) : un drapeau de CA qui permet à toute requête de spécifier son propre sujet, ou qui désactive la liaison SID à l'échelle du domaine.
- **Relais d'inscription** (ESC8, ESC11) : relais NTLM vers les points de terminaison d'inscription HTTP ou RPC de la CA.

## Pages

- **[Énumération](enumeration.md)** : trouver les CA, les modèles et les conditions vulnérables.
- **[Modèles vulnérables](vulnerable-templates.md)** : ESC1, ESC2, ESC3, ESC13, ESC15.
- **[Mappage de certificat](certificate-mapping.md)** : ESC9, ESC10, ESC14.
- **[Contrôle d'accès](access-control.md)** : ESC4, ESC5, ESC7.
- **[Configuration de la CA](ca-configuration.md)** : ESC6, ESC16.
- **[Relais vers AD CS](relay-to-adcs.md)** : ESC8, ESC11.
- **[Vol et pass-the-certificate](theft-and-pass-the-certificate.md)** : voler des certificats et s'authentifier avec.
- **[Schannel](schannel.md)** : s'authentifier avec un certificat via TLS/LDAPS plutôt que PKINIT.
- **[Certifried](certifried.md)** : un certificat machine pour l'identité d'un contrôleur de domaine via dNSHostName.
- **[Golden Certificate](golden-certificate.md)** : forger des certificats hors ligne avec une clé privée de CA volée.

## Références

- [SpecterOps: Certified Pre-Owned](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
- [Certipy (ly4k): AD CS enumeration and abuse](https://github.com/ly4k/Certipy)
- [Microsoft: Active Directory Certificate Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/)
