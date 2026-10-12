---
title: "Plateformes : attaquer les plateformes de monitoring et d'observabilité"
order: 2
description: "Les plateformes de monitoring sont des applications exposées sur le web qui stockent des identifiants pour le parc et exécutent souvent des commandes sur les hôtes surveillés par conception. Le flux d'attaque est cohérent entre elles : identifier et énumérer, obtenir l'accès via des identifiants par défaut ou faibles ou un contournement d'authentification, puis atteindre l'exécution de code ou récolter les identifiants qu'elles détiennent, transformant une seule plateforme en portée sur l'ensemble de l'environnement."
keywords:
  - monitoring platforms
  - zabbix
  - grafana
  - nagios
  - solarwinds
---

# Plateformes

Les plateformes de monitoring sont des applications web dotées de deux propriétés qui en font des cibles de premier choix : elles stockent des identifiants pour atteindre chaque système surveillé (identifiants d'équipements, chaînes SNMP, secrets de bases de données et de cloud), et beaucoup d'entre elles exécutent des commandes sur les hôtes surveillés en guise de fonctionnalité. Le flux d'attaque est le même d'un produit à l'autre : identifier et énumérer la version, obtenir l'accès via des identifiants par défaut ou faibles ou un contournement d'authentification, puis soit exécuter du code (via les fonctionnalités de commande/script de la plateforme elle-même ou une vulnérabilité connue) soit récolter les identifiants stockés. N'importe laquelle de ces voies convertit la compromission d'une seule plateforme en une large portée sur l'environnement.

## Sous-thèmes

- **[Zabbix](zabbix/index.md)** : items, scripts, l'agent, et bugs serveur connus.
- **[Nagios et Icinga](nagios-and-icinga/index.md)** : NRPE, injection de commandes, et chaînes Nagios XI.
- **[PRTG Network Monitor](prtg-network-monitor/index.md)** : exécution de code basée sur les notifications.
- **[Cacti](cacti/index.md)** : le RCE récurrent par SQLi et injection de commandes.
- **[LibreNMS et Observium](librenms-and-observium/index.md)** : injection de commandes et identifiants stockés.
- **[Grafana](grafana/index.md)** : traversée de chemin, SSRF des sources de données, et identifiants stockés.
- **[Prometheus](prometheus/index.md)** : interfaces exposées, SSRF, et abus d'exporters.
- **[SolarWinds Orion](solarwinds-orion/index.md)** : identifiants stockés et chaînes de RCE connues.

## Références

- [HackTricks: monitoring and management](https://book.hacktricks.xyz/network-services-pentesting)
- [CISA: exploitation of network management software](https://www.cisa.gov/news-events/cybersecurity-advisories)
