---
title: "Exposed interfaces and disclosure: reading an open Prometheus"
description: "Prometheus exposes, without authentication, the list of scrape targets, the full scrape configuration, the runtime flags, and all collected metrics. The targets and config map the internal environment and frequently embed credentials (basic-auth and bearer tokens in scrape configs), and the metrics themselves leak host inventory and secrets placed in labels."
keywords:
  - prometheus disclosure
  - api/v1/targets
  - status/config
  - scrape config
  - metrics labels
---

# Exposed interfaces and disclosure

An open Prometheus server is a rich, unauthenticated information source. The `/api/v1/targets` endpoint lists every scrape target, their addresses, labels, and health, a map of the monitored environment. The `/api/v1/status/config` endpoint returns the full `prometheus.yml` scrape configuration, which routinely embeds credentials: `basic_auth`, `bearer_token`, and authorization headers configured to scrape protected exporters and third-party endpoints are shown in the config. The `/api/v1/status/flags` and build-info endpoints disclose the runtime setup. And the metrics themselves (queried through `/api/v1/query`) leak host inventory, versions, and any secrets developers placed into metric labels. All of this without a credential, so a reachable Prometheus maps the estate and often hands over the credentials used to scrape it.

```bash
Pr=http://<target>:9090
curl -s $Pr/api/v1/targets | jq '.data.activeTargets[] | {scrapeUrl, labels}'   # the estate map
curl -s $Pr/api/v1/status/config | jq -r .data.yaml | grep -iE 'password|token|authorization|basic_auth'  # creds in config
curl -s $Pr/api/v1/status/flags; curl -s $Pr/api/v1/status/buildinfo
curl -s "$Pr/api/v1/query?query=up" | jq '.data.result[].metric'                 # metric labels
```

## Exploitation notes

- `/api/v1/status/config` is the credential prize: scrape configs embed `basic_auth`/`bearer_token` for protected targets, so the open config hands over those credentials; grep it for secret fields.
- `/api/v1/targets` maps the internal environment (every monitored address and its role), which is reconnaissance and a target list for pivoting.
- Metric labels sometimes carry secrets or internal detail developers did not consider exposed; query broadly and inspect labels.
- All unauthenticated by default; combine the recovered targets and credentials with the [SSRF](ssrf-and-federation-abuse.md) and [exporter](exporter-and-pushgateway-abuse.md) routes.

## References

- [Prometheus HTTP API](https://prometheus.io/docs/prometheus/latest/querying/api/)
- [Prometheus security model](https://prometheus.io/docs/operating/security/)
