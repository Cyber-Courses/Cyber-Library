---
title: "Platforms: attacking monitoring and observability platforms"
description: "Monitoring platforms are web-facing applications that store credentials for the estate and often execute commands on monitored hosts by design. The attack flow is consistent across them: fingerprint and enumerate, gain access through default or weak credentials or an auth bypass, then reach code execution or harvest the credentials they hold, turning one platform into reach across the environment."
keywords:
  - monitoring platforms
  - zabbix
  - grafana
  - nagios
  - solarwinds
---

# Platforms

Monitoring platforms are web applications with two properties that make them prime targets: they store credentials to reach every monitored system (device logins, SNMP strings, database and cloud secrets), and many of them run commands on monitored hosts as a feature. The attack flow is the same across products: fingerprint and enumerate the version, gain access through default or weak credentials or an authentication bypass, then either execute code (through the platform's own command/script features or a known vulnerability) or harvest the stored credentials. Any of these converts a single platform compromise into broad reach across the environment.

## Subtopics

- **[Zabbix](zabbix/index.md)**: items, scripts, the agent, and known server bugs.
- **[Nagios and Icinga](nagios-and-icinga/index.md)**: NRPE, command injection, and Nagios XI chains.
- **[PRTG Network Monitor](prtg-network-monitor/index.md)**: notification-based code execution.
- **[Cacti](cacti/index.md)**: the recurring SQLi and command-injection RCE.
- **[LibreNMS and Observium](librenms-and-observium/index.md)**: command injection and stored credentials.
- **[Grafana](grafana/index.md)**: path traversal, data-source SSRF, and stored credentials.
- **[Prometheus](prometheus/index.md)**: exposed interfaces, SSRF, and exporter abuse.
- **[SolarWinds Orion](solarwinds-orion/index.md)**: stored credentials and known RCE chains.

## References

- [HackTricks: monitoring and management](https://book.hacktricks.xyz/network-services-pentesting)
- [CISA: exploitation of network management software](https://www.cisa.gov/news-events/cybersecurity-advisories)
