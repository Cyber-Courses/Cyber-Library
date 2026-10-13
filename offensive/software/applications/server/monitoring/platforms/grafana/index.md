---
title: "Grafana: attacking the observability dashboards"
order: 6
description: "Grafana is a web dashboarding platform that connects to data sources holding credentials for databases and internal services. The surface is default and anonymous access, the unauthenticated plugin path-traversal that reads server files, server-side request forgery through data-source proxying with the stored data-source credentials, and known plugin and server vulnerabilities."
keywords:
  - grafana
  - path traversal
  - data source
  - ssrf
  - dashboards
---

# Grafana

Grafana is a widely deployed web application for dashboards and visualization, connecting to data sources (Prometheus, Elasticsearch, SQL databases, cloud APIs) that it stores credentials for. It is attacked several ways. Access comes from default credentials (`admin`/`admin`) and anonymous/viewer access where enabled. An unauthenticated plugin path-traversal has allowed reading arbitrary files from the Grafana server, recovering its configuration and secrets. The data-source proxy is a server-side request forgery primitive, reaching internal services from the Grafana host, and the stored data-source credentials give direct access to those databases and services. And Grafana and its plugins have other known vulnerabilities. A Grafana compromise therefore yields both server access and the credentials for everything it visualizes.

```bash
curl -sk https://<target>:3000/login | grep -ioE 'Grafana v[0-9.]+'   # fingerprint (default 3000)
```

## Subtopics

- **[Enumeration](enumeration.md)**: product and version fingerprinting.
- **[Authentication and anonymous access](authentication-and-anonymous-access.md)**: defaults and anonymous viewing.
- **[Path traversal file read](path-traversal-file-read.md)**: the unauthenticated plugin traversal.
- **[Data source SSRF and credentials](data-source-ssrf-and-credentials.md)**: proxy SSRF and stored secrets.
- **[Known exploits](known-exploits.md)**: other plugin and server vulnerabilities.

## References

- [Grafana documentation](https://grafana.com/docs/)
- [Grafana security advisories](https://grafana.com/security/security-advisories/)
