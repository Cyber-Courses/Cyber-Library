---
title: "Flooding and evasion: drowning and disrupting the log pipeline"
order: 3
description: "Syslog collectors have finite throughput and storage, so an attacker floods them with high-volume records to drown real events in noise, trigger rotation that ages out evidence, exhaust storage, or overwhelm the SIEM's ingestion and alerting. Flooding is used to hide activity within the surge and to degrade detection while an attack proceeds."
keywords:
  - log flooding
  - evasion
  - rotation
  - denial of service
  - siem
---

# Flooding and evasion

A syslog pipeline, collector, storage, and SIEM ingestion, has finite capacity, and an attacker abuses that to evade and disrupt. Sending a high volume of records drowns the genuine events of an attack in a flood of noise, so the real indicators are buried among thousands of benign or fabricated entries, making manual and automated review miss them. Sustained flooding forces log rotation and retention limits to age out or discard older records, destroying evidence of earlier activity. It can exhaust collector storage or saturate ingestion so that legitimate logs are dropped (many UDP collectors silently drop under load), creating blind spots. And it can overwhelm the SIEM's correlation and alerting, delaying or suppressing detection while the attack proceeds.

```bash
# high-volume record generation to the collector (UDP, no handshake to slow it)
yes '<34>flood filler event' | head -1000000 \
  | while read l; do printf '%s\n' "$l"; done | nc -u <collector> 514
# or a tight loop / parallel senders to sustain rate and force rotation/drops
for i in $(seq 1 50); do (while :; do logger -n <collector> -P 514 noise; done &) ; done
```

## Exploitation notes

- Two evasion effects: burying real events in noise (so they are missed in review and correlation) and forcing rotation/retention to discard older evidence; time the flood to cover the activity you want hidden.
- UDP collectors commonly drop silently under load, which creates genuine ingestion gaps, legitimate logs during the flood may never be stored, so the blind spot is real, not just noisy.
- Flooding is noisy by nature and may itself alert on volume anomalies; it is a trade, degrading detection and evidence at the cost of an obvious spike, so it suits short windows during a louder operation.
- Combine with [injection and forgery](injection-and-forgery.md) to make the flood also plant misleading events, and with [source spoofing](source-spoofing.md) to misattribute it.

## References

- [RFC 5426 (UDP syslog, delivery not guaranteed)](https://datatracker.ietf.org/doc/html/rfc5426)
- [HackTricks: syslog](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsyslog)
