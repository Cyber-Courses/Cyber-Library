---
order: 2
title: "Cryptographie faible : attaquer la négociation de chiffrement, de MAC et d'échange de clés SSH"
description: "SSH négocie ses algorithmes de chiffrement, de MAC et d'échange de clés, et un serveur qui propose encore des algorithmes faibles, chiffrements hérités, MAC basés sur MD5 ou tronqués, petits groupes Diffie-Hellman, permet à un attaquant positionné de sélectionner ou de rétrograder vers une session cassable. Ces faiblesses rendent possibles le déchiffrement ou l'altération d'un trafic que l'attaquant peut capturer ou relayer."
keywords:
  - crypto ssh
  - chiffrement
  - mac
  - échange de clés
  - downgrade
---

# Cryptographie faible

SSH commence chaque session en négociant les algorithmes d'échange de clés, de chiffrement et d'authentification des messages à partir des listes proposées par les deux parties. Si un serveur annonce encore des options faibles, la négociation (ou un attaquant positionné la forçant) peut aboutir à une combinaison cassable : chiffrements hérités ou cassés, MAC faibles ou tronqués, et groupes d'échange de clés petits ou défaillants. Cela n'a pas d'importance face à un observateur passif d'une session robuste, mais permet le déchiffrement, l'altération ou le downgrade pour un attaquant capable de capturer ou de se positionner sur la connexion, et ces faiblesses sont celles qui rendent possibles certaines attaques SSH de type machine-in-the-middle et sur le trafic.

```bash
# enumerate offered algorithms and grade them
nmap -p22 --script ssh2-enum-algos <target>
ssh -Q cipher; ssh -Q mac; ssh -Q kex          # what your client supports (for comparison)
# connect forcing a weak algorithm to test acceptance
ssh -c aes128-cbc -o MACs=hmac-md5 user@<target>
```

## Sous-thèmes

- **[Downgrade d'échange de clés](key-exchange-downgrade.md)** : forcer des groupes d'échange de clés faibles ou petits.
- **[Chiffrements faibles](weak-ciphers.md)** : faiblesses des chiffrements hérités et en mode CBC.
- **[Algorithmes MAC faibles](weak-mac-algorithms.md)** : protection d'intégrité cassée ou tronquée.

## Références

- [RFC 4253 (SSH transport layer)](https://datatracker.ietf.org/doc/html/rfc4253)
- [nmap ssh2-enum-algos](https://nmap.org/nsedoc/scripts/ssh2-enum-algos.html)
