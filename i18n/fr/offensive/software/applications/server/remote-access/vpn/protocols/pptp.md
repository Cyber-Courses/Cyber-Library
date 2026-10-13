---
order: 3
title: "PPTP : l'authentification MS-CHAPv2 cassée"
description: "PPTP (TCP 1723) s'authentifie avec MS-CHAPv2, dont la sécurité se réduit à une unique clé DES brute forçable en temps borné, de sorte qu'une négociation PPTP capturée est cassée pour récupérer le mot de passe ou la clé MPPE. PPTP est considéré comme cryptographiquement cassé, et une authentification capturée donne de façon fiable l'information d'identification hors ligne."
keywords:
  - pptp
  - ms-chapv2
  - chapcrack
  - des
  - mppe
---

# PPTP

PPTP (Point-to-Point Tunneling Protocol, TCP 1723 avec GRE pour les données) s'authentifie avec MS-CHAPv2, qui est cryptographiquement cassé. La réponse MS-CHAPv2 est dérivée du hash NT d'une façon qui se réduit, via l'usage par le protocole de trois opérations DES à structure connue, à la récupération d'une unique clé DES, brute forçable en temps borné quelle que soit la robustesse du mot de passe (la robustesse du mot de passe n'importe pas car l'attaque vise la clé dérivée du hash). Une négociation PPTP capturée est donc cassée hors ligne pour récupérer le mot de passe/hash NT et la clé de chiffrement MPPE, déchiffrant le tunnel. PPTP est déprécié précisément à cause de cela.

```bash
# capture a PPTP MS-CHAPv2 handshake (on-path or by prompting a connection)
tcpdump -i eth0 -w pptp.pcap tcp port 1723 or proto gre
# extract the MS-CHAPv2 challenge/response and crack to the DES key / NT hash
chapcrack parse -i pptp.pcap                     # extracts the handshake fields
# the reduced DES keyspace is brute-forced in bounded time (historically via cloud crackers)
```

## Notes d'exploitation

- L'attaque est indépendante de la robustesse du mot de passe : la structure de MS-CHAPv2 permet de réduire la réponse à une unique clé DES brute forçable en temps borné, de sorte que toute négociation capturée est cassable jusqu'au hash NT et à la clé MPPE.
- Une négociation capturée donne à la fois l'information d'identification (hash NT, utilisable pour du pass-the-hash ailleurs) et la clé pour déchiffrer le trafic du tunnel PPTP.
- La capture nécessite une position sur le chemin ou la capacité d'induire une connexion ; le canal de contrôle de PPTP (1723) et les données GRE sont tous deux interceptables.
- Traitez tout point de terminaison PPTP comme une cryptographie de niveau effectivement non authentifié ; il est déprécié et doit être supposé cassable dès qu'une négociation est observée.

## Outils

- [chapcrack](https://github.com/moxie0/chapcrack)

## Références

- [Cryptanalyse MS-CHAPv2 (Marlinspike/Hulton)](https://www.cloudcracker.com/blog/2012/07/29/cracking-ms-chap-v2/)
- [HackTricks: PPTP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-pptp)
