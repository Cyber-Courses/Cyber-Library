---
order: 3
title: "Plan de gestion : la toolstack Xen et XAPI"
description: "Xen est géré par une toolstack : xl/libxl sur Xen upstream, et XAPI sur XenServer et XCP-ng, qui ajoute une API XML-RPC/JSON-RPC à l'échelle du pool sur le port 443. L'accès à XAPI contrôle chaque hôte et VM d'un pool, le cycle de vie des VM, la console, les opérations sur disque et les commandes de l'hôte, de sorte que des identifiants XAPI, des jetons de session ou une faille de l'API donnent le contrôle à l'échelle du pool."
keywords:
  - xapi
  - xenserver
  - xcp-ng
  - toolstack
  - xe
---

# Plan de gestion

Xen upstream est piloté localement depuis dom0 avec la toolstack `xl`, mais les distributions managées, XenServer et XCP-ng, ajoutent XAPI, un service de gestion à l'échelle du pool exposant une API XML-RPC/JSON-RPC (sur 443) et piloté par la CLI `xe`. Un pool est un ensemble d'hôtes avec un coordinateur (maître) ; XAPI sur le coordinateur contrôle chaque hôte et VM du pool. Atteindre XAPI avec des identifiants, un jeton de session volé, ou via une faille de l'API équivaut au contrôle à l'échelle du pool : cycle de vie des VM, accès à la console, opérations de disque et de snapshot, et commandes de niveau hôte, sans qu'une évasion d'invité soit nécessaire.

## Atteindre et piloter XAPI

```bash
# the xe CLI against the pool coordinator
xe -s <host> -u root -pw <pw> vm-list
xe -s <host> -u root -pw <pw> host-list
# the API is XML-RPC/JSON-RPC over HTTPS; a session is opened then reused
curl -sk https://<host>/jsonrpc -d '{"method":"session.login_with_password","params":["root","<pw>"],"id":0,"jsonrpc":"2.0"}'
# act: run on a host, or operate on VM disks
xe -s <host> -u root -pw <pw> vm-import / vm-export ...   # export a VM (disk theft)
```

## Actions de contrôle

```text
With XAPI/pool access an attacker can:
- control every VM across the pool (start/stop/console/migrate)
- export or snapshot VM disks (VDIs) for offline data theft
- run host-level operations through host.* calls
- read pool configuration, users, and secrets
Compromising the pool coordinator extends control to all member hosts.
```

## Notes d'exploitation

- XAPI s'applique à l'échelle du pool : compromettre le XAPI du coordinateur contrôle chaque hôte et VM du pool, c'est donc la cible de gestion Xen à plus forte valeur.
- Les jetons de session sont des identifiants porteurs (bearer) une fois que `session.login_with_password` réussit ; un jeton volé agit sans se réauthentifier, et les opérations `vm-export`/VDI tirent les disques des invités sans évasion d'invité.
- `xl`/libxl sur Xen upstream est local à dom0 (pas d'API réseau), de sorte que la voie de gestion y est l'accès à dom0 lui-même, voir [Accès à l'hôte et shell](host-access-and-shell.md).
- Identifiez les versions de XenServer/XCP-ng et de XAPI pour tout avis relatif au plan de gestion.

## Références

- [XAPI et la CLI xe (XCP-ng/XenServer)](https://docs.xcp-ng.org/)
- [Toolstacks du projet Xen](https://wiki.xenproject.org/wiki/Choice_of_Toolstacks)
