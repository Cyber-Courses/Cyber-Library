---
title: "Internet exposure: remote-desktop reachability through vendor relays"
description: "Because remote-desktop tools connect out to a vendor relay, every running client is reachable from the internet without an inbound port, so the machine is exposed wherever the attacker can reach the relay. Combined with an enumerable ID and a weak or leaked password, this makes any installed client a remotely-connectable target."
keywords:
  - internet exposure
  - relay
  - reachability
  - device id
  - connectable
---

# Internet exposure

The relay architecture that makes these tools convenient also makes them broadly exposed. A client connects outward to the vendor's relay and waits, so an attacker who knows the device ID and password connects through that same relay from anywhere, without the target having any inbound port open. The machine is therefore effectively internet-reachable for remote control the moment the client runs, regardless of firewalls or NAT. Combined with [enumerable IDs](../authentication/id-based-access.md) and [weak or leaked passwords](../authentication/weak-passwords.md), this means any installed client is a remotely-connectable target, which is why these tools are a common initial-access and persistence vector, including for scam and ransomware operations.

```bash
# the machine need not be internet-facing: the client reaches out to the relay,
# and the attacker connects via the relay using ID + password from anywhere.
# target sources: enumerated IDs, and ID:password pairs from breaches/stealer logs.
```

## Exploitation notes

- The relay model defeats network segmentation as a control: a client on an internal machine is still reachable for remote control from the internet via the relay, so "not internet-facing" is not protection for these tools.
- The practical requirement is the ID plus password; infostealer logs commonly supply both for a specific victim, making exposure directly exploitable without scanning.
- This underlies the scam/support-fraud and ransomware use of these tools: a running client plus a socially-engineered or leaked password is remote control.
- Pair with [insecure defaults](insecure-defaults.md) (unattended access, weak policy) that make the reachable client open to connect.

## References

- [TeamViewer / AnyDesk relay connectivity](https://www.teamviewer.com/en/)
- [CISA: remote access software abuse](https://www.cisa.gov/news-events/cybersecurity-advisories)
