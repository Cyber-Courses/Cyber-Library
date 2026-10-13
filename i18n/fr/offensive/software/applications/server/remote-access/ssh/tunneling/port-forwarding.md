---
order: 3
title: "Redirection de port : franchir les frontières réseau avec SSH"
description: "La redirection de port locale (-L) et distante (-R) de SSH tunnelise un service TCP à travers la connexion SSH, ce qui permet à un attaquant d'atteindre un service sur un réseau qu'il ne peut pas router, ou d'exposer un service côté attaquant sur le réseau de l'hôte SSH. Cela fait le pont entre des frontières filtrées et segmentées en ne disposant que d'un point d'appui SSH."
keywords:
  - redirection de port
  - ssh -L
  - ssh -R
  - contournement de pare-feu
  - pivot
---

# Redirection de port

La redirection de port tunnelise un unique service TCP à travers une connexion SSH existante, en le déplaçant au-delà d'une frontière que l'attaquant ne pourrait autrement pas franchir. La redirection locale (`-L`) ouvre un port du côté de l'attaquant qui se connecte, via l'hôte SSH, à une destination que l'hôte SSH peut atteindre, de sorte qu'une base de données ou une interface d'administration sur un réseau interne devient accessible localement. La redirection distante (`-R`) fait l'inverse : elle ouvre un port sur l'hôte SSH qui se reconnecte vers un service côté attaquant, ce qui est utile pour exposer un payload ou un callback dans le réseau cible. Les deux ne nécessitent qu'une session SSH fonctionnelle vers le pivot.

```bash
# local forward: reach internal:3306 (not routable to you) via the pivot, on your 13306
ssh -L 13306:10.0.5.20:3306 user@pivot
mysql -h 127.0.0.1 -P 13306 -u app -p          # now hits the internal DB

# remote forward: expose your local:8000 as pivot:8000 inside the target network
ssh -R 8000:localhost:8000 user@pivot          # internal hosts can now reach your service

# keep it backgrounded and non-interactive
ssh -fN -L 13306:10.0.5.20:3306 user@pivot
```

## Notes d'exploitation

- La redirection locale est le pivot courant : elle ramène un service interne non routable (base de données, administration web, SMB) jusqu'à la machine de l'attaquant via l'hôte SSH, en contournant le pare-feu qui bloque l'accès direct.
- La redirection distante livre vers l'intérieur : exposez un listener de reverse shell ou un outil sur le réseau cible via le pivot, ou recevez un callback depuis un hôte qui peut atteindre le pivot mais pas l'attaquant.
- `GatewayPorts` sur le serveur SSH contrôle si un port redirigé à distance n'écoute que sur localhost ou sur toutes les interfaces du pivot ; ce dernier cas l'expose à tout le segment interne.
- Pour atteindre de nombreux hôtes/ports plutôt qu'un seul, utilisez plutôt la redirection dynamique, voir [Proxy SOCKS](socks-proxy.md) ; pour le multi-saut, voir [Abus d'hôte de rebond](jump-host-abuse.md).

## Références

- [OpenSSH: -L and -R](https://man.openbsd.org/ssh)
- [HackTricks: tunneling and port forwarding](https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding)
