---
title: "Prometheus : attaquer la plateforme de métriques"
order: 7
description: "Prometheus et son écosystème (exporteurs, Pushgateway, Alertmanager) sont non authentifiés par défaut, de sorte que leurs interfaces exposent les cibles de collecte, la configuration et des secrets, et que les fonctionnalités de requête du serveur lui-même (fédération, points de terminaison distants, cibles de sonde) sont des primitives de falsification de requête côté serveur. Les exporteurs et la Pushgateway ajoutent d'autres surfaces de divulgation et de pivot à travers le réseau de supervision."
keywords:
  - prometheus
  - exporters
  - pushgateway
  - ssrf
  - unauthenticated
---

# Prometheus

Prometheus est un système de collecte de métriques et d'alerte, et sa propriété de sécurité déterminante est qu'il est livré sans aucune authentification : le serveur Prometheus, les exporteurs qu'il collecte (node_exporter, blackbox_exporter et bien d'autres), la Pushgateway et Alertmanager exposent tous ouvertement leurs interfaces HTTP à moins que quelque chose d'externe ne les restreigne. Cela fait de l'écosystème avant tout une surface de divulgation d'information et de falsification de requête côté serveur plutôt qu'une cible classique d'authentification et de RCE. L'interface du serveur divulgue les cibles de collecte, la configuration de collecte complète (souvent avec des identifiants intégrés) et les drapeaux d'exécution ; ses fonctionnalités de fédération, de lecture/écriture distante et de sonde atteignent des adresses arbitraires depuis le serveur ; et les exporteurs et la Pushgateway divulguent des données d'hôte et fournissent des points de pivot et d'injection. Une pile Prometheus accessible cartographie l'environnement supervisé et atteint les services internes.

```bash
# the server and exporters answer openly by default
curl -s http://<target>:9090/api/v1/targets                    # scrape targets
curl -s http://<target>:9090/api/v1/status/config              # full config (may hold secrets)
curl -s http://<target>:9100/metrics | head                    # node_exporter
```

## Sous-thèmes

- **[Interfaces exposées et divulgation](exposed-interfaces-and-disclosure.md)** : cibles, configuration et secrets.
- **[SSRF et abus de fédération](ssrf-and-federation-abuse.md)** : atteindre les services internes depuis le serveur.
- **[Abus des exporteurs et de la Pushgateway](exporter-and-pushgateway-abuse.md)** : divulgation et injection par les exporteurs.

## Références

- [Documentation Prometheus](https://prometheus.io/docs/)
- [Sécurité de Prometheus](https://prometheus.io/docs/operating/security/)
