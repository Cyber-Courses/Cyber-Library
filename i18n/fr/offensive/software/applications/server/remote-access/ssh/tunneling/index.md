---
order: 5
title: "Tunneling : pivoter à travers SSH"
description: "Les fonctionnalités de redirection de SSH transforment un seul hôte atteignable en pivot réseau : les redirections de port locale et distante exposent des services de part et d'autre de la frontière réseau, la redirection dynamique fournit un proxy SOCKS pour un accès arbitraire plus loin, le chaînage d'hôtes de rebond atteint profondément les réseaux segmentés, et un agent transmis sur un hôte compromis est réutilisable pour aller plus loin."
keywords:
  - tunneling ssh
  - redirection de port
  - proxy socks
  - proxyjump
  - transmission d'agent
---

# Tunneling

SSH n'est pas qu'un shell ; ses fonctionnalités de redirection en font un outil de pivot, et un attaquant les utilise exactement comme le ferait un administrateur. Les redirections de port locale et distante relient un service de part et d'autre d'une frontière réseau que l'attaquant ne peut pas franchir autrement. La redirection dynamique dresse un proxy SOCKS qui achemine des outils arbitraires à travers l'hôte SSH vers les réseaux situés derrière lui. Le chaînage d'hôtes de rebond (ProxyJump) saute à travers les bastions profondément dans les environnements segmentés. Et la transmission d'agent SSH, lorsqu'un utilisateur transmet son agent vers un hôte contrôlé par l'attaquant, laisse un canal d'authentification utilisable pour aller plus loin en tant que cet utilisateur.

```bash
# the four building blocks
ssh -L 8080:internal:80 user@pivot             # local forward: reach internal:80 via pivot
ssh -R 9001:localhost:9001 user@pivot          # remote forward: expose attacker service on pivot
ssh -D 1080 user@pivot                          # dynamic: SOCKS proxy through pivot
ssh -J user@bastion user@internal               # jump through bastion to internal
```

## Sous-thèmes

- **[Redirection de port](port-forwarding.md)** : redirection locale et distante de part et d'autre des frontières.
- **[Proxy SOCKS](socks-proxy.md)** : redirection dynamique pour un accès arbitraire plus loin.
- **[Abus d'hôte de rebond](jump-host-abuse.md)** : chaînage à travers les bastions dans les réseaux segmentés.
- **[Détournement d'agent](agent-hijacking.md)** : réutilisation d'un agent SSH transmis.

## Références

- [OpenSSH port forwarding](https://man.openbsd.org/ssh#L)
- [HackTricks: SSH tunneling](https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding)
