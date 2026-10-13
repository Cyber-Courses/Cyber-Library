---
title: "Clés pré-partagées faibles : casser les PSK de VPN"
description: "Les VPN IPsec utilisant une clé pré-partagée pour l'authentification sont vulnérables lorsque cette clé est faible ou par défaut, en particulier en mode agressif IKE, qui envoie un hash de la PSK qu'un attaquant capture et casse hors ligne. Récupérer la PSK, combinée aux éventuelles informations d'identification XAUTH requises, authentifie l'attaquant auprès du VPN."
keywords:
  - clé pré-partagée
  - psk
  - mode agressif ike
  - cassage hors ligne
  - psk-crack
---

# Clés pré-partagées faibles

Les VPN IPsec (et L2TP/IPsec) authentifient souvent le tunnel avec une clé pré-partagée, et une PSK faible ou par défaut est récupérable. Le cas le plus marquant est le mode agressif IKE : pour économiser un aller-retour, le mode agressif envoie un hash calculé à partir de la PSK dans le premier échange, avant que la clé ne soit confirmée, de sorte qu'un attaquant qui initie une négociation en mode agressif capture ce hash et casse la PSK hors ligne avec une wordlist ou par brute force. Une fois la PSK récupérée, l'attaquant complète l'authentification de la Phase 1 (puis les éventuels nom d'utilisateur/mot de passe XAUTH, eux-mêmes souvent faibles), établissant le tunnel.

```bash
# detect and capture an aggressive-mode PSK hash
ike-scan -A -M -P psk.hash <target>             # -A aggressive, -P saves the hash
# crack the captured PSK offline
psk-crack -d wordlist.txt psk.hash
psk-crack -b 8 psk.hash                          # brute force up to length 8
# with the PSK (and XAUTH creds if required), connect
```

## Notes d'exploitation

- Le mode agressif est le facilitateur : il émet le hash dérivé de la PSK vers un initiateur non authentifié, de sorte que `ike-scan -A -P` le capture et que `psk-crack` récupère une clé faible hors ligne ; le mode principal ne divulgue pas cela, vérifiez donc quels modes la passerelle propose.
- Les PSK par défaut (valeurs par défaut des éditeurs, exemples de documentation, chaînes évidentes) et les clés courtes tombent rapidement ; alimentez la wordlist avec des termes propres à l'organisation.
- La PSK authentifie le tunnel mais de nombreux déploiements ajoutent XAUTH (un nom d'utilisateur/mot de passe) comme second facteur ; récupérez d'abord la PSK, puis attaquez XAUTH ([brute force de mot de passe](password-brute-force.md)).
- Une PSK récupérée plus XAUTH donne un tunnel complet vers le réseau interne ; voir [IPsec IKE](../protocols/ipsec-ike.md) pour les mécanismes du mode agressif.

## Outils

- [ike-scan / psk-crack](https://github.com/royhills/ike-scan)

## Références

- [HackTricks: IKE aggressive mode PSK](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
- [NIST SP 800-77 : IKE](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
