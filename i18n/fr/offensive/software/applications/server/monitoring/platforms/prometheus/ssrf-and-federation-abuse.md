---
order: 2
title: "Abus du SSRF et de la fédération : atteindre des services internes depuis Prometheus"
description: "Prometheus effectue des requêtes HTTP côté serveur pour la fédération, le remote-read/write et les cibles de sonde, et le blackbox exporter récupère des URL spécifiées par l'attaquant. Un attaquant capable d'influencer ces mécanismes transforme Prometheus en proxy de server-side request forgery qui atteint des services internes uniquement, des métadonnées cloud et des points de terminaison autrement inaccessibles depuis l'hôte Prometheus."
keywords:
  - prometheus ssrf
  - federation
  - remote read
  - blackbox exporter
  - metadata
---

# Abus du SSRF et de la fédération

Plusieurs fonctionnalités de Prometheus amènent le serveur à émettre des requêtes HTTP vers des adresses qui peuvent être influencées, ce qui constitue du server-side request forgery. La fédération (`/federate`) et le remote-read/remote-write font contacter d'autres points de terminaison par Prometheus ; les mécanismes de service-discovery HTTP et de sonde récupèrent des URL configurées ; et le blackbox exporter est explicitement conçu pour sonder une URL cible passée en paramètre, de sorte qu'un blackbox exporter accessible est une primitive SSRF générale (`/probe?target=<url>`). Un attaquant capable d'atteindre ces éléments (le blackbox exporter est souvent ouvert, et l'influence sur la configuration peut provenir d'une surface d'administration exposée) dirige Prometheus ou l'exporter vers des services internes uniquement, des points de terminaison de métadonnées d'instance cloud et d'autres adresses inaccessibles depuis l'attaquant, lisant ou agissant à travers la position réseau de l'hôte Prometheus.

```bash
# blackbox exporter as an SSRF proxy: fetch an internal/metadata URL via the exporter
curl -s 'http://<blackbox>:9115/probe?target=http://169.254.169.254/latest/meta-data/&module=http_2xx'
# the probe result/metrics reveal reachability and timing; some modules return body detail
# federation/remote endpoints: where config is influenceable, point them at internal targets
curl -s 'http://<target>:9090/federate?match[]={__name__=~".%2B"}'   # bulk metric exfil via federation
```

## Notes d'exploitation

- Le blackbox exporter est le SSRF le plus propre : `/probe?target=` récupère n'importe quelle URL depuis l'hôte de l'exporter, atteignant les services internes et les métadonnées cloud ; il est fréquemment exposé car les exporters ne sont pas authentifiés.
- La fédération et le remote-read/write atteignent d'autres points de terminaison depuis le serveur Prometheus ; là où vous pouvez influencer la configuration ou les cibles de collecte, ils deviennent un SSRF vers des adresses internes.
- Le gain est la position réseau de l'hôte Prometheus/exporter : les services internes et les points de terminaison de métadonnées inaccessibles depuis l'attaquant deviennent accessibles, y compris les identifiants cloud issus des métadonnées.
- À associer avec les [interfaces exposées](exposed-interfaces-and-disclosure.md) (pour trouver les cibles et la configuration) et l'[abus des exporters](exporter-and-pushgateway-abuse.md).

## Références

- [Prometheus blackbox exporter](https://github.com/prometheus/blackbox_exporter)
- [Prometheus federation](https://prometheus.io/docs/prometheus/latest/federation/)
