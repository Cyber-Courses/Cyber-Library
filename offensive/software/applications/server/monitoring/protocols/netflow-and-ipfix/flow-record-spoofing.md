---
title: "Flow record spoofing: forging exported flow records"
description: "Flow export is unauthenticated UDP, so an attacker who can send to the collector forges NetFlow/IPFIX records. Fabricated flows inject false conversations into the monitoring picture, can be used to hide real traffic in noise or misattribute activity, and can target bugs in the collector's template and record parsing, which has produced memory-corruption vulnerabilities."
keywords:
  - flow spoofing
  - netflow forgery
  - ipfix template
  - collector parsing
  - false traffic
---

# Flow record spoofing

NetFlow, sFlow, and IPFIX export records over UDP with no authentication, so a collector accepts flow records from anyone who can reach it. An attacker therefore forges records: crafting flow datagrams that describe conversations that never happened. Fabricated flows poison the monitoring picture, injecting false traffic to mislead capacity planning and security analytics, drowning or misattributing real flows, and planting evidence of connections that frame a host or hide activity in noise. IPFIX and NetFlow v9 are template-based (the exporter first sends a template describing the record layout, then data records referencing it), and parsing those attacker-controlled templates and records is a surface in itself: malformed templates and length fields have produced memory-corruption and denial-of-service vulnerabilities in collectors.

```bash
# forge a NetFlow/IPFIX datagram to the collector (UDP); craft header + records
# scapy has NetflowHeader/NetflowRecord layers; example sketch:
python3 - <<'PY'
from scapy.all import IP, UDP, send
from scapy.layers.netflow import NetflowHeader, NetflowHeaderV5, NetflowRecordV5
pkt = IP(dst="<collector>")/UDP(sport=1234, dport=2055)/ \
      NetflowHeader()/NetflowHeaderV5(count=1)/ \
      NetflowRecordV5(src="10.0.0.5", dst="10.0.0.9", dpkts=1000, doctets=1000000)
send(pkt, verbose=0)
PY
# crafted IPFIX/v9 templates with inconsistent field lengths probe collector parsing bugs
```

## Exploitation notes

- Spoofed flows mislead whatever is built on the data: capacity dashboards, baselines, and flow-based detection, so forgery is used to fabricate traffic (frame a host, hide real flows in noise) or to create false indicators.
- The collector trusts the exporter's source and template; port 2055/4739 (common NetFlow/IPFIX) over UDP has no authentication, so reachability is the only precondition.
- The template/record parser is a memory-safety surface: crafted IPFIX/NetFlow v9 templates with bad length or field counts have crashed or exploited collectors, so a flow listener is also an exploitation target, match the collector software to its advisories.
- Pair with [collector data harvesting](collector-data-harvesting.md): read the real map first, then forge flows that blend in or misdirect.

## Tools

- [Scapy (NetFlow layers)](https://scapy.net/)

## References

- [RFC 7011 (IPFIX)](https://datatracker.ietf.org/doc/html/rfc7011)
- [NetFlow v9 (RFC 3954)](https://datatracker.ietf.org/doc/html/rfc3954)
