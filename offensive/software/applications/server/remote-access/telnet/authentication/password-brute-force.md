---
title: "Password brute force: online attacks on the Telnet login"
description: "Telnet logins can be targeted by online password guessing, and some devices apply little rate limiting or lockout. If the session remains plaintext, capturing a real login is quieter than guessing; otherwise, use device-appropriate credential lists and conservative rates."
keywords:
  - telnet brute force
  - hydra
  - password spray
  - iot
  - cleartext
---

# Password brute force

Telnet logins can be targeted by online password guessing, and some equipment applies little account lockout or rate limiting. Device-appropriate wordlists are more relevant than generic lists, especially when paired with a username documented for the identified model. If the session remains plaintext and a capture position exists, sniffing a real login ([password sniffing](../traffic-interception/password-sniffing.md)) is quieter and more reliable than guessing.

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
