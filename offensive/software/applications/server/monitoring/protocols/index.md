---
title: "Protocols: attacking the monitoring telemetry protocols"
description: "The wire protocols that carry monitoring telemetry are attack surfaces in themselves: SNMP exposes device data and configuration and is often weakly authenticated, syslog ingestion trusts its input and is spoofable and injectable, and NetFlow/IPFIX collectors hold the network's traffic map. Each leaks information or lets an attacker poison what operators see."
keywords:
  - monitoring protocols
  - snmp
  - syslog
  - netflow
  - telemetry
---

# Protocols

Before the platforms, the telemetry protocols themselves are attackable. SNMP is the richest: a widely deployed, frequently weakly-authenticated protocol that reveals a device's entire state and, with write access, reconfigures it. Syslog is a trusting, usually unauthenticated ingestion path that an attacker spoofs, forges, and floods to poison the record operators and SIEMs rely on. And NetFlow/IPFIX collectors hold the map of who talks to whom across the network, valuable reconnaissance and a target for forged records. These are protocol-level attacks, independent of which platform consumes them.

## Subtopics

- **[SNMP](snmp/index.md)**: device data disclosure, weak community strings, and write-access abuse.
- **[Syslog](syslog/index.md)**: spoofing, injection, and flooding of the log stream.
- **[NetFlow and IPFIX](netflow-and-ipfix/index.md)**: harvesting and forging network flow telemetry.

## References

- [SNMP (RFC 1157, RFC 3416)](https://datatracker.ietf.org/doc/html/rfc3416)
- [Syslog (RFC 5424)](https://datatracker.ietf.org/doc/html/rfc5424)
- [IPFIX (RFC 7011)](https://datatracker.ietf.org/doc/html/rfc7011)
