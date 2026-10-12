---
title: "Récolte des données du collecteur : lire la carte du trafic du réseau"
description: "Un collecteur de flux stocke un enregistrement de chaque conversation réseau qui lui a été envoyée. Atteindre l'interface, l'API ou le stockage sous-jacent du collecteur, souvent une plateforme de supervision ou une base de données, permet à un attaquant d'interroger cette carte du trafic pour révéler les services internes, les relations client-serveur, les flux de données vers des destinations externes et les hôtes de grande valeur, une image complète de l'environnement sans le scanner."
keywords:
  - flow collector
  - traffic map
  - reconnaissance
  - netflow
  - relationships
---

# Récolte des données du collecteur

Le collecteur qui reçoit les exports de flux détient un enregistrement historique des conversations du réseau, et le lire constitue une reconnaissance passive de l'ensemble de l'environnement. Les collecteurs exposent les données via une interface web, une API ou une base de données sous-jacente (et font souvent partie d'une plateforme de supervision telle que SolarWinds, d'un appareil d'analyse de flux, ou d'une pile Elastic/Grafana), de sorte que l'accès à l'un d'eux permet à un attaquant d'interroger la carte du trafic. Cette carte révèle quels services internes existent et qui les utilise (relations client vers serveur), quels hôtes communiquent avec Internet et où (flux de données et canaux potentiels d'exfiltration ou de C2 déjà présents), les hôtes les plus actifs et les plus connectés (systèmes probablement de grande valeur), et la segmentation (ou son absence) entre les zones. C'est la structure et les relations du réseau étalées, obtenues sans envoyer un seul paquet aux hôtes eux-mêmes.

```bash
# the access path depends on the collector/platform; examples:
#  - a flow-analysis web UI / API: query top talkers, conversations, and flows
#  - a backing store (e.g. Elasticsearch holding flow docs): query it directly
curl -s 'http://<collector>:9200/flows-*/_search?size=100&q=*' | jq '.hits.hits[]._source'
#  - nfdump/nfcapd files on the collector host, read with nfdump
nfdump -R /var/cache/nfdump -o extended 'dst port 3389'   # e.g. who uses RDP internally
```

## Notes d'exploitation

- La carte du trafic est une reconnaissance que vous ne pouvez obtenir par scan : elle montre les relations et l'usage réels dans le temps, y compris les services internes, les destinations externes, et quels hôtes sont centraux, guidant le ciblage et le pivot.
- Les flux vers des destinations externes peuvent révéler une exfiltration ou un command-and-control existants, et les services utilisés (RDP, SSH, ports de base de données) indiquent où se déplacer ensuite et quels identifiants rechercher.
- L'accès passe par ce qui expose ou stocke les flux, une plateforme de supervision, un appareil d'analyse, un index Elastic, ou des fichiers `nfcapd` bruts sur l'hôte du collecteur, de sorte que compromettre la plateforme de collecte fournit la carte.
- Ceci est en lecture seule et discret ; associez-le à la [falsification d'enregistrements de flux](flow-record-spoofing.md) lorsque l'objectif est de tromper plutôt que d'observer.

## Outils

- [nfdump (NetFlow capture/analysis)](https://github.com/phaag/nfdump)

## Références

- [RFC 7011 (IPFIX)](https://datatracker.ietf.org/doc/html/rfc7011)
- [SiLK flow analysis](https://tools.netsa.cert.org/silk/)
