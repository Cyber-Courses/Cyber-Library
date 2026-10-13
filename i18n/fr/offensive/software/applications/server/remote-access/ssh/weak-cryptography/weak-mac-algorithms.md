---
order: 2
title: "Algorithmes MAC faibles : protection d'intégrité SSH cassée"
description: "SSH protège chaque paquet avec un code d'authentification de message, et les serveurs qui proposent des MAC faibles, basés sur MD5, tronqués à 96 bits, ou en modes autres que encrypt-then-MAC, affaiblissent la garantie d'intégrité. Un attaquant positionné exploite les MAC faibles ou tronqués pour altérer ou forger des paquets, sapant la protection qui empêche la manipulation de session."
keywords:
  - mac faible
  - hmac-md5
  - mac tronqué
  - encrypt-then-mac
  - intégrité
---

# Algorithmes MAC faibles

SSH ajoute un code d'authentification de message (MAC) à chaque paquet afin que toute altération soit détectée. La robustesse de cette garantie dépend du MAC négocié, et les vieux serveurs en proposent de faibles : MAC basés sur MD5 (`hmac-md5`), MAC tronqués à 96 bits (`hmac-*-96`), et MAC utilisés dans la construction plus faible encrypt-and-MAC (plutôt que encrypt-then-MAC, `*-etm`). Les MAC faibles ou tronqués réduisent le travail nécessaire pour forger ou altérer des paquets, de sorte qu'un attaquant positionné les utilise pour manipuler la session, sapant la protection d'intégrité qui bloque sinon les attaques d'injection et de modification.

```bash
# which MACs are offered?
nmap -p22 --script ssh2-enum-algos <target> | grep -A20 mac_algorithms
# force a weak MAC to confirm acceptance
ssh -o MACs=hmac-md5 user@<target>
ssh -o MACs=hmac-sha1-96 user@<target>        # 96-bit truncated
```

## Notes d'exploitation

- La liste `mac_algorithms` proposée signale la faiblesse : `hmac-md5*`, toute troncature `*-96`, et l'absence de variantes `-etm` (encrypt-then-MAC) indiquent une intégrité plus faible.
- Les MAC tronqués et MD5 réduisent l'effort nécessaire pour forger un tag valide pour un texte chiffré altéré ; combinés à un chiffrement CBC faible et à une position on-path, cela rend possible la manipulation de session plutôt que la seule observation passive.
- Comme les faiblesses de chiffrement et de KEX, les MAC faibles importent surtout à un attaquant capable de capturer ou de relayer le trafic ; ils identifient aussi un build SSH obsolète.
- Ces trois pages de crypto faible se cumulent : un serveur proposant des MAC faibles, des chiffrements CBC et de petits groupes KEX est un candidat aux attaques cryptographiques de session et au machine-in-the-middle SSH, voir [SSH MITM](../mitm-and-trust/ssh-mitm.md).

## Références

- [RFC 4253: data integrity](https://datatracker.ietf.org/doc/html/rfc4253)
- [OpenSSH encrypt-then-MAC](https://www.openssh.com/txt/release-6.2)
