---
title: "Brute force: wordlist attacks against the community string"
description: "SNMP has no account lockout and the community string is the only credential, so brute force with a wordlist is practical. onesixtyone sends rapid UDP probes across many strings and hosts at once, and snmp-check or hydra confirm hits. Sweeping a community wordlist across a subnet quickly finds every weakly-stringed agent."
keywords:
  - snmp brute force
  - onesixtyone
  - wordlist
  - no lockout
  - udp 161
---

# Brute force

Because SNMP has no concept of an account or lockout and the community string is the sole credential, brute forcing it is straightforward and fast. `onesixtyone` is purpose-built: it fires UDP SNMP requests across a list of community strings (and a list of hosts) at high rate and reports which strings elicit a response, making it practical to sweep a whole subnet against a community wordlist in seconds. A recovered string is then confirmed and used with the net-snmp tools. The lack of lockout and the connectionless UDP transport mean the only real limit is packet loss, so retries matter.

```bash
# sweep community strings (and optionally many hosts) with onesixtyone
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <target>
onesixtyone -c comm.txt -i hosts.txt                     # many hosts at once
# confirm a hit and start reading
snmp-check -c <found> <target>
# hydra also brute-forces SNMP (slower, but integrates with other services)
hydra -P comm.txt -s 161 <target> snmp
```

## Exploitation notes

- `onesixtyone` is the fast path: it is UDP-rate-driven and sweeps both strings and hosts, so it is ideal for finding every weakly-stringed agent on a subnet in one pass.
- Account for UDP loss: use retries and reasonable rate; a missed response is not a failed string, so re-test apparent misses before concluding.
- There is no lockout to trip, so a large wordlist is viable, but a device may still rate-limit or drop under flood; tune the rate if responses dry up.
- A found string drives [enumeration](../enumeration/index.md); separately brute-force for a read-write string, as it is often different and enables [write access](../write-access.md).

## Tools

- [onesixtyone](https://github.com/trailofbits/onesixtyone)
- [snmp-check](https://gitlab.com/kalilinux/packages/snmpcheck)

## References

- [HackTricks: SNMP brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
