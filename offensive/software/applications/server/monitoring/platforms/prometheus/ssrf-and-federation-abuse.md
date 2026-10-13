---
title: "SSRF and federation abuse: reaching internal services from Prometheus"
order: 2
description: "Prometheus makes server-side HTTP requests for federation, remote-read/write, and probe targets, and the blackbox exporter fetches attacker-specified URLs. An attacker who can influence these turns Prometheus into a server-side request forgery proxy that reaches internal-only services, cloud metadata, and otherwise-unreachable endpoints from the Prometheus host."
keywords:
  - prometheus ssrf
  - federation
  - remote read
  - blackbox exporter
  - metadata
---

# SSRF and federation abuse

Several Prometheus features make the server issue HTTP requests to addresses that can be influenced, which is server-side request forgery. Federation (`/federate`) and remote-read/remote-write have Prometheus contact other endpoints; the HTTP service-discovery and probe mechanisms fetch configured URLs; and the blackbox exporter is explicitly designed to probe a target URL passed as a parameter, so a reachable blackbox exporter is a general SSRF primitive (`/probe?target=<url>`). An attacker who can reach these (the blackbox exporter is often open, and config influence may come from an exposed management surface) directs Prometheus or the exporter at internal-only services, cloud instance-metadata endpoints, and other addresses unreachable from the attacker, reading or acting through the Prometheus host's network position.

```bash
# blackbox exporter as an SSRF proxy: fetch an internal/metadata URL via the exporter
curl -s 'http://<blackbox>:9115/probe?target=http://169.254.169.254/latest/meta-data/&module=http_2xx'
# the probe result/metrics reveal reachability and timing; some modules return body detail
# federation/remote endpoints: where config is influenceable, point them at internal targets
curl -s 'http://<target>:9090/federate?match[]={__name__=~".%2B"}'   # bulk metric exfil via federation
```

## Exploitation notes

- The blackbox exporter is the cleanest SSRF: `/probe?target=` fetches any URL from the exporter's host, reaching internal services and cloud metadata; it is frequently exposed because exporters are unauthenticated.
- Federation and remote-read/write reach other endpoints from the Prometheus server; where you can influence config or scrape targets, they become SSRF to internal addresses.
- The payoff is the Prometheus/exporter host's network position: internal services and metadata endpoints unreachable from the attacker become reachable, including cloud credentials from metadata.
- Pair with [exposed interfaces](exposed-interfaces-and-disclosure.md) (to find targets and config) and [exporter abuse](exporter-and-pushgateway-abuse.md).

## References

- [Prometheus blackbox exporter](https://github.com/prometheus/blackbox_exporter)
- [Prometheus federation](https://prometheus.io/docs/prometheus/latest/federation/)
