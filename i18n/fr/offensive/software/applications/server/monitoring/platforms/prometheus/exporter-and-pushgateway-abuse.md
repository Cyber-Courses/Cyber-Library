---
title: "Abus des exporteurs et de la Pushgateway : divulgation, pivot et injection"
order: 3
description: "Les exporteurs Prometheus exposent des données d'hôte détaillées sans authentification (node_exporter révèle le système en profondeur), le textfile collector lit des fichiers modifiables par l'attaquant pour les transformer en métriques, le blackbox exporter est un proxy SSRF, et une Pushgateway ouverte accepte des métriques arbitraires poussées. Ensemble, ils divulguent des informations d'hôte, offrent un pivot et permettent à un attaquant d'injecter de fausses métriques pour tromper la supervision."
keywords:
  - node_exporter
  - textfile collector
  - pushgateway
  - blackbox
  - metric injection
---

# Abus des exporteurs et de la Pushgateway

Les exporteurs Prometheus et la Pushgateway constituent chacun leur propre surface. `node_exporter` (par défaut 9100) expose sans authentification de nombreux détails sur l'hôte : CPU, mémoire, disques, réseau, systèmes de fichiers montés, unités en cours d'exécution et utilisateurs, profilant le système pour un attaquant. Son textfile collector lit des fichiers `.prom` depuis un répertoire et publie leur contenu sous forme de métriques, de sorte que si ce répertoire est modifiable par l'attaquant (via un autre point d'ancrage), le contenu injecté apparaît dans Prometheus, et cela peut servir de canal d'exfiltration de données ou d'injection. Le blackbox exporter est le proxy SSRF traité séparément. Et la Pushgateway (par défaut 9091) accepte les métriques poussées par quiconque peut l'atteindre et les conserve pour la collecte, de sorte qu'une Pushgateway ouverte permet à un attaquant d'injecter des métriques arbitraires pour fabriquer des données, masquer des signaux réels, ou déclencher ou supprimer des alertes. Ensemble, ces éléments divulguent des données d'hôte, fournissent des pivots et permettent à un attaquant d'empoisonner le tableau de la supervision.

```bash
# node_exporter: deep host profiling, unauthenticated
curl -s http://<target>:9100/metrics | grep -E 'node_(uname|filesystem|network|systemd)' | head
# Pushgateway: inject arbitrary metrics (anyone who can reach it)
echo 'injected_metric 1' | curl -s --data-binary @- http://<target>:9091/metrics/job/fake
# textfile collector: if the collector directory is writable (via another foothold),
# dropped *.prom files are published as metrics (injection / exfil channel)
```

## Notes d'exploitation

- `node_exporter` est une divulgation d'hôte approfondie et non authentifiée : il profile le système (matériel, systèmes de fichiers, réseau, unités, utilisateurs) à des fins de ciblage, et est presque toujours accessible là où il est déployé.
- Une Pushgateway ouverte est une attaque contre l'intégrité de la supervision : les métriques injectées fabriquent des données et manipulent les alertes (créer de fausses alertes ou supprimer les vraies en écrasant des séries), utile pour tromper les opérateurs pendant une opération.
- Le textfile collector transforme une écriture sur le système de fichiers (depuis un autre point d'ancrage sur l'hôte de l'exporteur) en injection de métriques ou en canal dissimulé ; il n'est pas distant en lui-même mais s'enchaîne avec un accès à l'hôte.
- Le SSRF du blackbox exporter est traité dans [SSRF et abus de fédération](ssrf-and-federation-abuse.md) ; ensemble, les exporteurs constituent la surface de divulgation et de pivot de l'écosystème.

## Références

- [node_exporter](https://github.com/prometheus/node_exporter)
- [Prometheus Pushgateway](https://github.com/prometheus/pushgateway)
