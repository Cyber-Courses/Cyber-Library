---
title: "Monitoring: attacking monitoring and observability infrastructure"
description: "Monitoring systems are a high-value target because they hold credentials for and reach into the whole estate they watch. The area splits into the telemetry protocols (SNMP, syslog, NetFlow/IPFIX) and the platforms that collect and visualize them (Zabbix, Nagios, PRTG, Cacti, LibreNMS, Grafana, Prometheus, SolarWinds), each a path to information, credentials, and code execution."
keywords:
  - monitoring
  - snmp
  - zabbix
  - grafana
  - observability
---

# Monitoring

Monitoring and observability infrastructure is a disproportionately valuable target. To do its job it must reach into every system it watches, so it holds credentials for the estate (SNMP strings, device logins, database and cloud secrets), maps the internal network, and often runs commands on monitored hosts by design. Compromising a monitoring platform therefore yields reconnaissance, stored credentials, and frequently code execution across many systems at once. The area divides into two layers: the telemetry **protocols** that agents and collectors speak, and the **platforms** that collect, store, and visualize that telemetry.

```bash
# find monitoring surfaces on a host or network
nmap -sU -p161,162 <target>                    # SNMP agent / traps
nmap -p443,80,3000,9090,10050,10051,5666,8086 -sV <target>  # web UIs, Zabbix, NRPE, etc.
```

## Subtopics

- **[Protocols](protocols/index.md)**: SNMP, syslog, and NetFlow/IPFIX.
- **[Platforms](platforms/index.md)**: Zabbix, Nagios/Icinga, PRTG, Cacti, LibreNMS, Grafana, Prometheus, SolarWinds.

## References

- [HackTricks: network services pentesting](https://book.hacktricks.xyz/network-services-pentesting)
- [MITRE ATT&CK: network sniffing and valid accounts](https://attack.mitre.org/techniques/T1040/)
