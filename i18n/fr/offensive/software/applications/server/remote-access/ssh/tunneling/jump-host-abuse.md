---
order: 1
title: "Abus d'hôte de rebond : chaînage à travers les bastions dans les réseaux segmentés"
description: "La prise en charge des hôtes de rebond par SSH (ProxyJump / -J) chaîne de façon transparente les connexions à travers un ou plusieurs hôtes intermédiaires, c'est ainsi que les bastions donnent accès aux réseaux segmentés. Un attaquant ayant accès à ou à travers un bastion utilise le même mécanisme pour atteindre des hôtes internes autrement isolés, et un bastion compromis expose chaque réseau qu'il relie."
keywords:
  - proxyjump
  - hôte de rebond
  - bastion
  - -J
  - multi-saut
---

# Abus d'hôte de rebond

Les organisations placent devant les réseaux segmentés des hôtes bastion (de rebond) : la seule façon d'entrer dans le segment protégé est de passer en SSH par le bastion. Le `ProxyJump` (`-J`) de SSH automatise ce saut, chaînant la connexion à travers un ou plusieurs intermédiaires de façon transparente. Un attaquant utilise le mécanisme identique de manière offensive : avec un accès au bastion (identifiants, une clé) ou la capacité de s'authentifier à travers lui, il atteint les hôtes internes qu'il relie, et un bastion entièrement compromis est une passerelle vers chaque segment réseau qu'il connecte, en plus d'un trésor d'identifiants et de clés qui y transitent.

```bash
# chain through a bastion to an internal host in one command
ssh -J user@bastion user@10.0.9.15
# multi-hop through two jumps
ssh -J user@bastion1,user@bastion2 user@deep-internal
# make it persistent in ~/.ssh/config for tooling to use the chain
#   Host internal
#     HostName 10.0.9.15
#     ProxyJump user@bastion
```

## Notes d'exploitation

- Le bastion est la cible de grande valeur : il est atteignable par conception et relie vers le segment protégé, donc le compromettre (via les attaques d'authentification SSH) ouvre tout ce qui se trouve derrière lui.
- Un bastion voit les identifiants et les clés de quiconque saute à travers lui ; un pied-à-terre là permet de les capturer ou de les réutiliser (et la transmission d'agent à travers lui est directement abusable, voir [Détournement d'agent](agent-hijacking.md)).
- `ProxyJump` se chaîne proprement pour les environnements multi-couches ; combinez avec un [proxy SOCKS](socks-proxy.md) sur le dernier saut pour exécuter des outils arbitraires contre le segment le plus profond.
- L'accès à l'hôte de rebond plus un agent transmis est une escalade classique : le bastion utilise l'agent de l'utilisateur qui saute pour s'authentifier plus loin, donc l'attaquant sur le bastion s'authentifie en tant que cet utilisateur vers le prochain saut.

## Références

- [OpenSSH: ProxyJump](https://man.openbsd.org/ssh_config#ProxyJump)
- [HackTricks: jump hosts](https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding)
