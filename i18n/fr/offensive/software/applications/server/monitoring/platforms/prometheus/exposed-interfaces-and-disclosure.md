---
title: "Interfaces exposées et divulgation : lire un Prometheus ouvert"
order: 1
description: "Prometheus expose, sans authentification, la liste des cibles de collecte, la configuration de collecte complète, les drapeaux d'exécution et toutes les métriques collectées. Les cibles et la configuration cartographient l'environnement interne et contiennent fréquemment des identifiants (basic-auth et jetons bearer dans les configurations de collecte), et les métriques elles-mêmes divulguent l'inventaire des hôtes et les secrets placés dans les étiquettes."
keywords:
  - prometheus disclosure
  - api/v1/targets
  - status/config
  - scrape config
  - metrics labels
---

# Interfaces exposées et divulgation

Un serveur Prometheus ouvert est une riche source d'information non authentifiée. Le point de terminaison `/api/v1/targets` liste chaque cible de collecte, leurs adresses, leurs étiquettes et leur état de santé, soit une carte de l'environnement supervisé. Le point de terminaison `/api/v1/status/config` renvoie la configuration de collecte complète `prometheus.yml`, qui contient régulièrement des identifiants : les champs `basic_auth`, `bearer_token` et les en-têtes d'autorisation configurés pour collecter des exporteurs protégés et des points de terminaison tiers apparaissent dans la configuration. Les points de terminaison `/api/v1/status/flags` et build-info divulguent la configuration d'exécution. Et les métriques elles-mêmes (interrogées via `/api/v1/query`) divulguent l'inventaire des hôtes, les versions et tout secret que les développeurs ont placé dans les étiquettes de métriques. Tout cela sans identifiant, de sorte qu'un Prometheus accessible cartographie le parc et livre souvent les identifiants utilisés pour le collecter.

```bash
Pr=http://<target>:9090
curl -s $Pr/api/v1/targets | jq '.data.activeTargets[] | {scrapeUrl, labels}'   # the estate map
curl -s $Pr/api/v1/status/config | jq -r .data.yaml | grep -iE 'password|token|authorization|basic_auth'  # creds in config
curl -s $Pr/api/v1/status/flags; curl -s $Pr/api/v1/status/buildinfo
curl -s "$Pr/api/v1/query?query=up" | jq '.data.result[].metric'                 # metric labels
```

## Notes d'exploitation

- `/api/v1/status/config` est le butin en matière d'identifiants : les configurations de collecte contiennent des champs `basic_auth`/`bearer_token` pour les cibles protégées, de sorte que la configuration ouverte livre ces identifiants ; filtrez-la à la recherche des champs de secrets.
- `/api/v1/targets` cartographie l'environnement interne (chaque adresse supervisée et son rôle), ce qui constitue de la reconnaissance et une liste de cibles pour le pivot.
- Les étiquettes de métriques contiennent parfois des secrets ou des détails internes que les développeurs ne pensaient pas exposés ; interrogez largement et inspectez les étiquettes.
- Tout cela est non authentifié par défaut ; combinez les cibles et identifiants récupérés avec les voies [SSRF](ssrf-and-federation-abuse.md) et [exporteur](exporter-and-pushgateway-abuse.md).

## Références

- [API HTTP de Prometheus](https://prometheus.io/docs/prometheus/latest/querying/api/)
- [Modèle de sécurité de Prometheus](https://prometheus.io/docs/operating/security/)
