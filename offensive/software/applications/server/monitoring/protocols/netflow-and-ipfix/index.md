---
title: "NetFlow and IPFIX: attacking network flow telemetry"
order: 3
description: "Flow export protocols (NetFlow, sFlow, IPFIX) summarize network conversations and send them to a collector. That data is a map of who talks to whom across the environment, valuable reconnaissance when harvested from an exposed collector, and the unauthenticated export path lets an attacker forge flow records to inject false traffic or mislead monitoring."
keywords:
  - netflow
  - ipfix
  - sflow
  - flow collector
  - telemetry
---

# NetFlow and IPFIX

Flow-export protocols, Cisco NetFlow, sFlow, and the IETF-standard IPFIX, have routers and switches summarize the conversations passing through them (source, destination, ports, protocol, byte and packet counts) and export those records to a collector over UDP. For an attacker the flow data is a ready-made map of the network's communication: who talks to whom, on which services, and how much, which is high-value reconnaissance obtained without touching the hosts. Harvesting that map from an exposed collector or its storage reveals the environment's structure and relationships. And because the export path is unauthenticated UDP, an attacker who can send to the collector forges flow records to inject false traffic, hide real flows, or mislead the capacity and security monitoring built on the data.

## Subtopics

- **[Collector data harvesting](collector-data-harvesting.md)**: reading the traffic map from a collector.
- **[Flow record spoofing](flow-record-spoofing.md)**: forging exported flow records.

## References

- [RFC 7011 (IPFIX)](https://datatracker.ietf.org/doc/html/rfc7011)
- [Cisco NetFlow overview](https://www.cisco.com/c/en/us/products/ios-nx-os-software/ios-netflow/index.html)
