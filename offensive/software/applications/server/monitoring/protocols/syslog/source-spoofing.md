---
title: "Source spoofing: forging the origin of syslog records"
order: 1
description: "UDP syslog has no authentication and the recorded source of an event is taken from the packet, so an attacker spoofs the source IP to attribute forged log entries to another host. This frames other systems, plants misleading evidence, and, because the collector trusts the apparent origin, poisons host-attributed detection and correlation."
keywords:
  - source spoofing
  - udp syslog
  - ip spoofing
  - forged origin
  - attribution
---

# Source spoofing

UDP syslog carries no authentication, and the collector attributes each record to the source it appears to come from, the packet's source IP and the hostname field in the message. An attacker on a network path that reaches the collector spoofs the UDP source address, so forged records are logged as if they originated from another host. This lets an attacker attribute fabricated events to a chosen system, framing it, planting misleading evidence, or triggering host-specific detection and response against an innocent machine. Because so much SIEM logic keys on the source host, spoofed-origin records corrupt correlation and can steer an investigation entirely.

```bash
# spoof the UDP source address of a syslog packet (needs raw-socket capability)
# scapy example: forge a record that appears to come from 10.0.0.5
python3 - <<'PY'
from scapy.all import IP, UDP, Raw, send
pkt = IP(src="10.0.0.5", dst="<collector>")/UDP(sport=514, dport=514)/ \
      Raw(load="<34>1 2025-01-01T00:00:00Z victimhost sshd - - - Accepted password for admin")
send(pkt, verbose=0)
PY
# the message's own hostname field is also attacker-controlled and often trusted
```

## Exploitation notes

- Two attribution channels are forgeable: the UDP source IP (spoofed at the packet level) and the hostname field inside the syslog message; collectors may trust either, so set both to the host you want to frame.
- The attack needs network reach to the collector with the ability to send spoofed UDP (raw sockets, and a path that does not filter spoofed source addresses); flat internal networks commonly allow it.
- Spoofed-origin records poison host-attributed detection and correlation, useful to mislead responders, generate false alerts against a target, or make real activity look like it came from elsewhere.
- Combine with [injection and forgery](injection-and-forgery.md) to control the content as well as the origin of the planted records.

## References

- [RFC 5426 (Syslog over UDP)](https://datatracker.ietf.org/doc/html/rfc5426)
- [HackTricks: syslog](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsyslog)
