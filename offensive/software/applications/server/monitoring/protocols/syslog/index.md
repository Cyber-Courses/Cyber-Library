---
title: "Syslog: attacking log ingestion"
description: "Syslog carries log events to collectors and SIEMs, usually over unauthenticated UDP 514, and the pipeline trusts what it receives. That trust is the attack surface: spoofing the source of records, injecting forged and newline-split entries to poison the record and exploit downstream parsers, and flooding the collector to drown real events or disrupt alerting."
keywords:
  - syslog
  - udp 514
  - log injection
  - spoofing
  - siem
---

# Syslog

Syslog is the standard protocol for shipping log events from systems and devices to a central collector or SIEM, classically over UDP 514 with no authentication and no integrity protection (TCP and TLS variants exist but are far less common). The entire monitoring and detection pipeline downstream trusts those records, and that trust is the attack surface. An attacker on the network spoofs the source address of records to attribute forged logs to other hosts, injects crafted and newline-split entries to forge events and poison the parsers and correlation rules the SIEM depends on, and floods the collector to drown real signal or disrupt the pipeline. The goal is usually to mislead operators, hide activity, or exploit a downstream log-processing sink.

```bash
# send an arbitrary syslog record (UDP 514)
logger -n <collector> -P 514 -d "test from attacker"
echo '<34>1 2025-01-01T00:00:00Z host app - - - forged event' | nc -u -w1 <collector> 514
```

## Subtopics

- **[Source spoofing](source-spoofing.md)**: forging the origin of log records.
- **[Injection and forgery](injection-and-forgery.md)**: forged entries and downstream parser abuse.
- **[Flooding and evasion](flooding-and-evasion.md)**: drowning and disrupting the pipeline.

## References

- [RFC 5424 (Syslog protocol)](https://datatracker.ietf.org/doc/html/rfc5424)
- [HackTricks: 514 syslog](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsyslog)
