---
title: "Password brute force: online attacks on the Telnet login"
description: "Telnet's cleartext login is brute-forceable, and the devices that run it often apply no lockout or rate limiting, so guessing is practical. Because capture is easier, sniffing a cleartext login beats guessing where a position exists, but brute force with device-appropriate wordlists remains effective against exposed Telnet."
keywords:
  - telnet brute force
  - hydra
  - password spray
  - iot
  - cleartext
---

# Password brute force

Telnet's plaintext login can be brute-forced, and the equipment that still exposes Telnet frequently has no account lockout or rate limiting, which makes online guessing viable where it would be impractical elsewhere. Device-appropriate wordlists (IoT/router default and common passwords) are effective, especially combined with a username derived from the device defaults. That said, because Telnet is cleartext, if a capture position exists, sniffing a real login ([password sniffing](../traffic-interception/password-sniffing.md)) is quieter and surer than guessing.

```bash
hydra -l admin -P passwords.txt telnet://<target> -t 4 -f
hydra -L users.txt -P iot-passwords.txt telnet://<target> -t 4
nmap -p23 --script telnet-brute --script-args userdb=u.txt,passdb=p.txt <target>
```

## Exploitation notes

- Tune the wordlist to the device: IoT/router default-and-common lists outperform generic human-password lists against embedded Telnet, and pair them with the device's default username.
- Many Telnet devices lack lockout, so brute force that would lock a normal account can run freely; still keep threads modest to avoid overwhelming flimsy device stacks (some crash).
- Prefer capture over guessing when on-path: a cleartext login sniff yields the exact credential with no attempts logged, see [password sniffing](../traffic-interception/password-sniffing.md).
- A hit is typically administrative on the device and often reused; test it elsewhere.

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [nmap telnet-brute](https://nmap.org/nsedoc/scripts/telnet-brute.html)

## References

- [HackTricks: Telnet brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
