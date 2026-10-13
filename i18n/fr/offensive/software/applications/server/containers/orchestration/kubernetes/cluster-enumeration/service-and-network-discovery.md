---
title: "Découverte des services et du réseau : cartographier les services, les endpoints et le réseau des pods"
order: 5
description: "Kubernetes donne par défaut à chaque pod un DNS de cluster et un réseau de pods plat. Un attaquant résout les noms de services, liste les services et les endpoints via l'API ou le DNS, et scanne les CIDR des pods et des services pour trouver des charges de travail, des bases de données et des API internes accessibles, qui sont fréquemment non authentifiées sous l'hypothèse que seuls les pairs du cluster peuvent les atteindre."
keywords:
  - dns kubernetes
  - services endpoints
  - réseau des pods
  - cluster.local
  - scan réseau
---

# Découverte des services et du réseau

Par défaut, un pod peut résoudre le DNS du cluster et atteindre chaque autre pod et service sur un réseau plat, car Kubernetes n'applique aucune isolation réseau en l'absence de NetworkPolicy. Cela rend la découverte des services et du réseau très productive : les services internes supposent que seuls des pairs intra-cluster peuvent les atteindre et sont souvent non authentifiés, de sorte que le simple fait de les trouver et de s'y connecter donne accès à des bases de données, des caches, des files de messages et des API internes.

## Découvrir les services via l'API et le DNS

```bash
# services and their cluster IPs/ports via the API
kapi /api/v1/services | python3 -c 'import sys,json;[print(f"{s[\"metadata\"][\"namespace\"]}/{s[\"metadata\"][\"name\"]} {s[\"spec\"].get(\"clusterIP\")} {[p.get(\"port\") for p in s[\"spec\"].get(\"ports\",[])]}") for s in json.load(sys.stdin)["items"]]'
# endpoints map services to the actual pod IPs behind them
kapi /api/v1/endpoints
# cluster DNS: SRV records enumerate ports; names follow a fixed scheme
nslookup -type=srv _https._tcp.kubernetes.default.svc.cluster.local 2>/dev/null
getent hosts <service>.<namespace>.svc.cluster.local
```

Les noms DNS de services sont déterministes (`<service>.<namespace>.svc.cluster.local`), de sorte que même sans accès en lecture à l'API, deviner des noms courants (`postgres`, `redis`, `elasticsearch`, `vault`) et les résoudre permet de trouver des backends.

## Scanner les réseaux des pods et des services

```bash
# find the pod CIDR from the local address and routes
ip -4 addr; ip route
# sweep the pod network for reachable services (adjust the range)
for i in $(seq 1 254); do (nc -z -w1 10.244.0.$i 6379 2>/dev/null && echo "10.244.0.$i:6379 open") & done; wait
# common internal ports: 6379 redis, 5432 postgres, 9200 elasticsearch, 8080 apis
```

## Notes d'exploitation

- Les endpoints sont plus utiles que les services pour le mouvement latéral, car ils listent les véritables IP des pods ; atteindre directement une IP de pod contourne les hypothèses d'accès propres à un service.
- Les services internes sont fréquemment non authentifiés par conception ; un Redis, un Elasticsearch ou une API HTTP interne accessible ne nécessite souvent aucun identifiant, faisant de la découverte réseau un gain direct d'accès aux données.
- Lorsqu'une NetworkPolicy bloque effectivement le trafic, voir [Contournement de NetworkPolicy](../network/networkpolicy-bypass.md) ; l'hypothèse du réseau plat ne tient que jusqu'à ce qu'une politique existe.
- Alimentez les pods accessibles découverts dans [Pivotement de pod à pod](../lateral-movement/pod-to-pod-pivoting.md).

## Outils

- [kube-hunter (reconnaissance réseau de cluster)](https://github.com/aquasecurity/kube-hunter)
- [nmap](https://nmap.org/)

## Références

- [Kubernetes : DNS pour les services et les pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Kubernetes : services et endpoints](https://kubernetes.io/docs/concepts/services-networking/service/)
