---
title: "Énumération des algorithmes : détecter l'exposition cryptographique faible de SSH"
description: "Un serveur SSH annonce ses algorithmes d'échange de clés, de clé d'hôte, de chiffrement et de MAC pris en charge durant le handshake non authentifié. Les énumérer et les évaluer révèle si le serveur propose des options faibles ou héritées, ce qui à la fois identifie une version obsolète et met en évidence les attaques concrètes de rétrogradation et de cryptographie faible disponibles contre lui."
keywords:
  - énumération des algorithmes
  - ssh2-enum-algos
  - kex ciphers macs
  - cryptographie faible
  - handshake
---

# Énumération des algorithmes

Une partie du handshake SSH, envoyée avant l'authentification, est la liste des algorithmes pris en charge par chaque partie pour l'échange de clés, la signature de clé d'hôte, le chiffrement et le MAC. Lire les listes du serveur vous indique exactement ce qu'il va accepter, et les évaluer par rapport aux références modernes identifie les options faibles ou héritées. Cela sert deux objectifs : identifier l'ancienneté et la configuration du serveur (les anciennes versions proposent de larges ensembles hérités), et énumérer les attaques de cryptographie faible disponibles, puisque vous ne pouvez pas forcer un algorithme que le serveur ne propose pas.

```bash
# enumerate the four algorithm lists the server offers
nmap -p22 --script ssh2-enum-algos <target>
# interpret: look for weak entries in each list
#   kex:     diffie-hellman-group1-sha1, *-sha1, small GEX groups
#   cipher:  *-cbc, arcfour*, 3des, des
#   mac:     hmac-md5*, *-96 (truncated), non -etm
#   hostkey: ssh-rsa (SHA-1) only, ssh-dss (DSA)
ssh -vv user@<target> 2>&1 | grep -iE 'kex:|cipher:|mac:'   # what gets negotiated
```

## Notes d'exploitation

- Associez chaque liste proposée à sa faiblesse : KEX `group1`/SHA-1 à la [rétrogradation de l'échange de clés](../weak-cryptography/key-exchange-downgrade.md), `-cbc`/`arcfour`/`3des` aux [chiffrements faibles](../weak-cryptography/weak-ciphers.md), `hmac-md5`/`-96` aux [algorithmes MAC faibles](../weak-cryptography/weak-mac-algorithms.md).
- Les clés d'hôte `ssh-dss` (DSA) et `ssh-rsa` limité à SHA-1 indiquent un serveur ancien ; DSA est désactivé dans les versions modernes d'OpenSSH, donc sa présence est un fort signal d'ancienneté.
- Vous ne pouvez attaquer que les algorithmes que le serveur propose, donc cette énumération est le préalable qui transforme les pages de cryptographie faible, de la théorie en une liste de cibles concrètes.
- Un ensemble d'algorithmes pleinement moderne (KEX curve25519, chiffrements ChaCha20/AES-GCM, MAC ETM) signifie que la surface cryptographique est fermée ; pivotez plutôt vers les vulnérabilités d'authentification et d'implémentation.

## Outils

- [nmap ssh2-enum-algos](https://nmap.org/nsedoc/scripts/ssh2-enum-algos.html)
- [ssh-audit](https://github.com/jtesta/ssh-audit)

## Références

- [RFC 4253 : négociation des algorithmes](https://datatracker.ietf.org/doc/html/rfc4253)
- [Guides de durcissement ssh-audit](https://www.ssh-audit.com/)
