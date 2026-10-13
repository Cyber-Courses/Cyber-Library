---
order: 1
title: "Downgrade d'échange de clés : forcer un échange de clés SSH faible"
description: "L'échange de clés SSH établit les clés de session, et un serveur proposant des méthodes faibles, de petits groupes Diffie-Hellman (group1/1024 bits) ou des échanges hérités, peut être rétrogradé vers eux. Un attaquant positionné capable d'influencer la négociation ou qui enregistre le handshake cible ces échanges plus faibles pour attaquer les clés de session résultantes."
keywords:
  - échange de clés
  - diffie-hellman
  - group1
  - downgrade
  - kex
---

# Downgrade d'échange de clés

L'échange de clés SSH (KEX) dérive les clés symétriques de session et authentifie la clé d'hôte du serveur. Sa robustesse dépend de la méthode négociée : les échanges modernes basés sur les courbes sont robustes, mais les serveurs qui proposent encore `diffie-hellman-group1-sha1` (un groupe fixe de 1024 bits) ou d'autres KEX hérités présentent une cible plus faible. Comme les deux parties choisissent dans les listes proposées, un serveur qui annonce des groupes faibles peut finir par les utiliser, et un attaquant positionné qui influence l'ensemble des algorithmes négociés, ou qui se contente d'enregistrer un handshake ayant utilisé un groupe faible, attaque l'accord de clés pour récupérer ou affaiblir les clés de session.

```bash
# which KEX methods does the server offer?
nmap -p22 --script ssh2-enum-algos <target> | grep -A15 kex_algorithms
# force a weak KEX to confirm the server accepts it
ssh -o KexAlgorithms=diffie-hellman-group1-sha1 user@<target>
ssh -o KexAlgorithms=diffie-hellman-group-exchange-sha1 user@<target>
```

## Notes d'exploitation

- Le signal est la liste `kex_algorithms` proposée par le serveur : la présence de `group1` (1024 bits), d'échanges basés sur SHA-1, ou du GEX avec de petites tailles de groupe marque une négociation faible pouvant être forcée.
- Un groupe fixe de 1024 bits est à la portée d'attaques par précalcul bien dotées contre les paramètres partagés ; l'impact pratique est de rendre possible le déchiffrement d'une session capturée qui l'a utilisé.
- Cela se combine avec une capture ou une position on-path : forcer ou observer un KEX faible est le travail préparatoire pour attaquer la session, plutôt qu'une rupture autonome.
- L'ensemble plus large de faiblesses (chiffrements, MAC) aggrave cela ; un serveur proposant un KEX faible propose généralement aussi des chiffrements et des MAC faibles.

## Références

- [RFC 4253: key exchange](https://datatracker.ietf.org/doc/html/rfc4253)
- [Logjam and weak DH groups](https://weakdh.org/)
