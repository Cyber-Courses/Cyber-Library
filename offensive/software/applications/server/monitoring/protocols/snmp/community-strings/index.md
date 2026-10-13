---
title: "Community strings: obtaining SNMP access"
order: 1
description: "On SNMPv1 and v2c the community string is the only credential: a read string permits querying, a read-write string permits reconfiguration. Strings are obtained the same ways weak credentials always are, by trying the near-universal defaults, guessing weak or predictable values, and brute-forcing with a wordlist, and a working string is the whole access."
keywords:
  - community string
  - public
  - private
  - default
  - brute force
---

# Community strings

On SNMPv1 and v2c the community string is the entire access control: there is no username, and the string travels in cleartext. Conventionally `public` is the read-only string and `private` is the read-write string, but any value can be configured. Obtaining a string is therefore the goal, and it is obtained like any weak credential: by trying the near-universal defaults, by guessing weak or predictable values (often derived from the organization or hostname), and by brute-forcing against a wordlist. A read string yields full enumeration and disclosure; a read-write string additionally yields reconfiguration. Read and write access can use different strings, so finding a read string does not imply write, and vice versa.

```bash
# quickest: test the obvious strings
for c in public private community manager; do
  snmpget -v2c -c $c -t1 -r0 <target> 1.3.6.1.2.1.1.1.0 2>/dev/null && echo "WORKS: $c"; done
```

## Subtopics

- **[Default community strings](default-community-strings.md)**: the near-universal and vendor defaults.
- **[Weak community strings](weak-community-strings.md)**: predictable and organization-derived strings.
- **[Brute force](brute-force.md)**: wordlist attacks against the string.

## References

- [HackTricks: SNMP community strings](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
- [SecLists SNMP community strings](https://github.com/danielmiessler/SecLists)
