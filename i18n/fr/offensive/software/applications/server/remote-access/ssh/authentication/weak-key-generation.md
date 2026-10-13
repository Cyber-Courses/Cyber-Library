---
title: "Génération de clés faibles : clés SSH prévisibles issues de générateurs défaillants"
description: "Les clés SSH générées avec une entropie insuffisante ou par un outillage défaillant sont prévisibles ou factorisables, donc la clé privée peut être dérivée sans la voler. La faille d'entropie d'OpenSSL sous Debian a produit un espace de clés réduit et énumérable dont les clés sont forcées contre authorized_keys, et les appareils embarqués à faible entropie et les clés d'usine partagées répètent les clés sur plusieurs unités."
keywords:
  - clé faible
  - debian openssl
  - clé prévisible
  - faible entropie
  - clé factorisée
---

# Génération de clés faibles

L'authentification à clé publique suppose que la clé privée ne peut pas être dérivée, mais des clés mal générées brisent cette hypothèse. L'archétype est la faille d'entropie d'OpenSSL sous Debian : pendant une période, les clés générées sur les systèmes affectés puisaient dans un pool d'entropie minuscule, donc seules quelques centaines de milliers de clés distinctes par type/taille existaient, et l'ensemble complet était précalculé. Toute clé publique d'un système affecté est mise en correspondance avec sa clé privée par recherche dans une table. Au-delà de cela, les appareils embarqués et IoT à faible entropie au premier démarrage génèrent des clés prévisibles, et beaucoup embarquent des clés d'usine identiques sur toutes les unités, donc récupérer la clé d'un appareil ouvre chaque pair.

```bash
# identify the key type/size in use (from a captured public key or host key)
ssh-keygen -lf authorized_keys                 # fingerprint and bit length
# Debian weak keys: match a target public key against the precomputed set
#   download the known weak-key set for the type/size, grep the target's blob
grep -f <(cut -d' ' -f2 target.pub) debian_ssh_rsa_2048/*.pub   # find the matching private key
# then authenticate with the recovered private key
ssh -i debian_ssh_rsa_2048/<match> user@<target>
# embedded/shared factory keys: extract from one device's firmware, reuse on peers
```

## Notes d'exploitation

- Le cas des clés faibles Debian est une recherche, pas un cassage : des jeux de clés précalculés existent, donc une clé publique (clé d'hôte ou entrée `authorized_keys`) d'un système affecté donne directement la clé privée.
- Les clés courtes ou héritées (RSA 512/1024 bits, DSA) sont en outre faibles face à la factorisation ou à des défauts intrinsèques ; `ssh-keygen -lf` révèle la taille pour en juger.
- Les appareils embarqués réutilisent souvent une clé d'usine sur toute une gamme de produits ; l'extraire du firmware (ou d'une unité) authentifie à tous, et la même clé sert fréquemment à la fois de clé d'hôte et de clé client de confiance.
- Cela complète l'[Exposition de clé publique](public-key-exposure.md) : là vous volez la clé, ici vous la dérivez ou la réutilisez ; les deux aboutissent à une authentification en tant que propriétaire.

## Outils

- [g0tmi1k debian-ssh weak key sets](https://github.com/g0tmi1k/debian-ssh)
- [ssh-keygen](https://man.openbsd.org/ssh-keygen)

## Références

- [Debian OpenSSL predictable PRNG advisory](https://www.debian.org/security/2008/dsa-1571)
- [HackTricks: SSH weak keys](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
