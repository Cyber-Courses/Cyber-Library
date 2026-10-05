---
title: "Collector data harvesting: reading the network's traffic map"
description: "A flow collector stores a record of every network conversation it was sent. Reaching the collector's interface, API, or backing store, often a monitoring platform or a database, lets an attacker query that traffic map to reveal internal services, client-server relationships, data flows to external destinations, and high-value hosts, a complete picture of the environment without scanning it."
keywords:
  - flow collector
  - traffic map
  - reconnaissance
  - netflow
  - relationships
---

# Collector data harvesting

The collector that receives flow exports holds a historical record of the network's conversations, and reading it is passive reconnaissance of the whole environment. Collectors expose the data through a web UI, an API, or a backing database (and are often part of a monitoring platform such as SolarWinds, a flow-analysis appliance, or an Elastic/Grafana stack), so access to any of those lets an attacker query the traffic map. That map reveals which internal services exist and who uses them (client-to-server relationships), which hosts talk to the internet and where (data flows and potential exfiltration or C2 channels already present), the busiest and most-connected hosts (likely high-value systems), and the segmentation (or lack of it) between zones. It is the network's structure and relationships laid out, obtained without sending a single packet to the hosts themselves.

```bash
# the access path depends on the collector/platform; examples:
#  - a flow-analysis web UI / API: query top talkers, conversations, and flows
#  - a backing store (e.g. Elasticsearch holding flow docs): query it directly
curl -s 'http://<collector>:9200/flows-*/_search?size=100&q=*' | jq '.hits.hits[]._source'
#  - nfdump/nfcapd files on the collector host, read with nfdump
nfdump -R /var/cache/nfdump -o extended 'dst port 3389'   # e.g. who uses RDP internally
```

## Exploitation notes

- The traffic map is reconnaissance you cannot get by scanning: it shows real relationships and usage over time, including internal services, external destinations, and which hosts are central, guiding targeting and pivoting.
- Flows to external destinations can reveal existing exfiltration or command-and-control, and the services in use (RDP, SSH, database ports) point at where to move next and what credentials to seek.
- Access comes through whatever fronts or stores the flows, a monitoring platform, an analysis appliance, an Elastic index, or raw `nfcapd` files on the collector host, so compromising the collecting platform yields the map.
- This is read-only and quiet; pair with [flow record spoofing](flow-record-spoofing.md) when the goal is to mislead rather than observe.

## Tools

- [nfdump (NetFlow capture/analysis)](https://github.com/phaag/nfdump)

## References

- [RFC 7011 (IPFIX)](https://datatracker.ietf.org/doc/html/rfc7011)
- [SiLK flow analysis](https://tools.netsa.cert.org/silk/)
