---
title: "Prometheus: attacking the metrics platform"
description: "Prometheus and its ecosystem (exporters, Pushgateway, Alertmanager) are unauthenticated by default, so their interfaces expose scrape targets, configuration, and secrets, and the server's own request features (federation, remote endpoints, probe targets) are server-side request forgery primitives. The exporters and Pushgateway add further disclosure and pivot surfaces across the monitoring network."
keywords:
  - prometheus
  - exporters
  - pushgateway
  - ssrf
  - unauthenticated
---

# Prometheus

Prometheus is a metrics collection and alerting system, and its defining security property is that it ships with no authentication: the Prometheus server, the exporters it scrapes (node_exporter, blackbox_exporter, and many others), the Pushgateway, and Alertmanager all expose their HTTP interfaces openly unless something external restricts them. That makes the ecosystem primarily an information-disclosure and server-side-request-forgery surface rather than a classic auth-and-RCE target. The server's interface leaks scrape targets, the full scrape configuration (often with embedded credentials), and runtime flags; its federation, remote-read/write, and probe features reach arbitrary addresses from the server; and the exporters and Pushgateway disclose host data and provide pivot and injection points. A reachable Prometheus stack maps the monitored environment and reaches internal services.

```bash
# the server and exporters answer openly by default
curl -s http://<target>:9090/api/v1/targets                    # scrape targets
curl -s http://<target>:9090/api/v1/status/config              # full config (may hold secrets)
curl -s http://<target>:9100/metrics | head                    # node_exporter
```

## Subtopics

- **[Exposed interfaces and disclosure](exposed-interfaces-and-disclosure.md)**: targets, config, and secrets.
- **[SSRF and federation abuse](ssrf-and-federation-abuse.md)**: reaching internal services from the server.
- **[Exporter and Pushgateway abuse](exporter-and-pushgateway-abuse.md)**: exporter disclosure and injection.

## References

- [Prometheus documentation](https://prometheus.io/docs/)
- [Prometheus security](https://prometheus.io/docs/operating/security/)
