---
order: 5
title: "Vol et pass-the-certificate : voler et utiliser des certificats"
description: "Voler des certificats et des clés privées sur des hôtes Windows (magasins utilisateur et machine, clés protégées par DPAPI, PFX exportés) et s'authentifier avec via PKINIT et Schannel (pass-the-certificate), y compris récupérer le hash NT."
keywords:
  - pass the certificate
  - vol de certificat
  - PKINIT
  - DPAPI
  - PFX
---

# Vol et pass-the-certificate

Toute attaque par certificat ne requiert pas une mauvaise configuration. Les certificats et leurs clés privées existent déjà sur les hôtes, dans les magasins utilisateur et machine, et un certificat d'authentification valide est un identifiant durable : il n'est pas invalidé par un changement de mot de passe et dure souvent un an ou plus. En **voler** un et s'**authentifier avec** (pass-the-certificate) est une primitive propre de déplacement latéral et de persistance.

## Voler des certificats

- **Magasins de certificats utilisateur et machine** : les certificats exportables avec leurs clés privées peuvent être extraits en PFX, depuis la mémoire ou le magasin, sur un hôte compromis.
- **Clés protégées par DPAPI** : les clés privées sont protégées par [DPAPI](../credentials/dpapi-secrets.md) ; avec la clé maîtresse de l'utilisateur (ou la clé de sauvegarde du domaine), elles se déchiffrent hors ligne.
- **Certificats machine** : le certificat propre d'un hôte s'authentifie en tant que **compte machine**, utile pour les silver tickets et le RBCD.

```bash
# Mimikatz: export certificates and keys from the stores
crypto::capi & crypto::certificates /export
# Certipy: collect certificates and DPAPI-protected keys from a host
certipy cert -export ...
```

## Pass-the-certificate

Un certificat volé ou émis s'authentifie de deux manières :

- **PKINIT** (Kerberos) : échanger le certificat contre un **TGT**, puis se déplacer avec le ticket :

```bash
certipy auth -pfx victim.pfx -dc-ip <dc>          # PKINIT -> TGT (and NT hash via UnPAC)
gettgtpkinit.py -cert-pfx victim.pfx example.local/victim victim.ccache
```

- **Schannel** (LDAPS/HTTPS) : se lier à LDAP via TLS avec le certificat client, utile lorsque PKINIT est indisponible mais que le mappage Schannel est activé ([ESC10](certificate-mapping.md)) :

```bash
certipy auth -pfx victim.pfx -ldap-shell -dc-ip <dc>
```

## Récupérer le hash NT

PKINIT renvoie le hash NT du compte aux côtés du TGT via [UnPAC-the-hash](../kerberos/unpac-the-hash.md), de sorte qu'un certificat volé devient un **hash réutilisable**, convertissant un certificat à durée limitée en un identifiant qui alimente le [pass-the-hash](../ntlm/pass-the-hash.md) indéfiniment.

## Notes d'exploitation

- Un certificat survit à la **réinitialisation du mot de passe** du propriétaire, un certificat volé ou inscrit par l'attaquant constitue donc de la persistance : il continue de s'authentifier jusqu'à son expiration ou sa révocation.
- Les certificats machine s'authentifient en tant que `HOST$` et alimentent les [silver tickets](../kerberos/forged-tickets.md) et le [RBCD](../kerberos/delegation/resource-based-constrained.md).
- Inscrire un certificat pour *vous-même* maintenant, pendant que vous avez un accès, est une démarche de persistance délibérée (persistance de compte par certificat), indépendante de tout ESC.

## Outils

- **Certipy** (`cert`, `auth`, `-ldap-shell`) : export, authentification PKINIT et Schannel.
- **Mimikatz** (`crypto::certificates`) : exporter des certificats et des clés depuis les magasins Windows.
- **PKINITtools** : PKINIT explicite et UnPAC.

## Références

- [Certipy (ly4k): auth, shadow, and certificate theft](https://github.com/ly4k/Certipy)
- [PKINITtools (dirkjanm): PKINIT and UnPAC](https://github.com/dirkjanm/PKINITtools)
- [SpecterOps: Certified Pre-Owned (theft and persistence)](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
