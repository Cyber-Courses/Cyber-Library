---
title: "Exporter and Pushgateway abuse: disclosure, pivot, and injection"
order: 3
description: "Prometheus exporters expose detailed host data unauthenticated (node_exporter reveals the system in depth), the textfile collector reads attacker-writable files into metrics, the blackbox exporter is an SSRF proxy, and an open Pushgateway accepts arbitrary pushed metrics. Together they disclose host information, provide a pivot, and let an attacker inject false metrics to mislead monitoring."
keywords:
  - node_exporter
  - textfile collector
  - pushgateway
  - blackbox
  - metric injection
---

# Exporter and Pushgateway abuse

The Prometheus exporters and the Pushgateway are each their own surface. `node_exporter` (default 9100) exposes extensive host detail unauthenticated, CPU, memory, disks, network, mounted filesystems, running units, and users, profiling the system for an attacker. Its textfile collector reads `.prom` files from a directory and publishes their contents as metrics, so if that directory is attacker-writable (via another foothold), injected content appears in Prometheus, and it can be a data-exfiltration or injection channel. The blackbox exporter is the SSRF proxy covered separately. And the Pushgateway (default 9091) accepts metrics pushed by anyone who can reach it and holds them for scraping, so an open Pushgateway lets an attacker inject arbitrary metrics to fabricate data, mask real signals, or trigger or suppress alerts. Together these disclose host data, provide pivots, and let an attacker poison the monitoring picture.

```bash
# node_exporter: deep host profiling, unauthenticated
curl -s http://<target>:9100/metrics | grep -E 'node_(uname|filesystem|network|systemd)' | head
# Pushgateway: inject arbitrary metrics (anyone who can reach it)
echo 'injected_metric 1' | curl -s --data-binary @- http://<target>:9091/metrics/job/fake
# textfile collector: if the collector directory is writable (via another foothold),
# dropped *.prom files are published as metrics (injection / exfil channel)
```

## Exploitation notes

- `node_exporter` is unauthenticated deep host disclosure: it profiles the system (hardware, filesystems, network, units, users) for targeting, and is almost always reachable where it is deployed.
- An open Pushgateway is a monitoring-integrity attack: injected metrics fabricate data and manipulate alerts (create false alerts or suppress real ones by overwriting series), useful to mislead operators during an operation.
- The textfile collector turns a filesystem write (from another foothold on the exporter host) into metric injection or a covert channel; it is not remote by itself but chains with host access.
- The blackbox exporter's SSRF is covered in [SSRF and federation abuse](ssrf-and-federation-abuse.md); together the exporters are the ecosystem's disclosure and pivot surface.

## References

- [node_exporter](https://github.com/prometheus/node_exporter)
- [Prometheus Pushgateway](https://github.com/prometheus/pushgateway)
