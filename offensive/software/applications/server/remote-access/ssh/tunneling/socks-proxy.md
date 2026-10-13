---
title: "SOCKS proxy: arbitrary onward access through dynamic forwarding"
order: 2
description: "SSH dynamic forwarding (-D) runs a SOCKS proxy on the attacker's side that routes any TCP connection through the SSH host into the networks it can reach. Combined with proxychains, this lets arbitrary tools, scanners, and clients operate against an internal network from a single SSH foothold, without forwarding each port individually."
keywords:
  - socks proxy
  - ssh -D
  - dynamic forwarding
  - proxychains
  - pivot
---

# SOCKS proxy

Dynamic forwarding (`-D`) turns an SSH connection into a general-purpose pivot: instead of tunnelling one fixed destination like local forwarding, it runs a SOCKS proxy on the attacker's machine that accepts connections to any address and routes each through the SSH host. Paired with a SOCKS-aware wrapper such as `proxychains`, this lets arbitrary tools, port scanners, web clients, database clients, exploit modules, operate against the entire network the SSH host can reach, from a single foothold and without setting up a forward per service.

```bash
# stand up a SOCKS proxy through the pivot
ssh -fN -D 1080 user@pivot
# point tools at it via proxychains (set socks5 127.0.0.1 1080 in proxychains.conf)
proxychains nmap -sT -Pn -p 22,445,3389 10.0.5.0/24
proxychains curl http://10.0.5.20/
proxychains mysql -h 10.0.5.30 -u app -p
# some tools accept a SOCKS proxy natively
curl --socks5 127.0.0.1:1080 http://10.0.5.20/
```

## Exploitation notes

- Dynamic forwarding is the efficient pivot when you need to reach many hosts and ports on the internal network, versus local forwarding's one-destination-per-tunnel; one `-D` covers the whole reachable network.
- Use TCP connect scans through the proxy (`nmap -sT -Pn`): SOCKS carries TCP, so SYN scans and techniques needing raw sockets or UDP do not work through it.
- `proxychains` (with `proxy_dns` considered) routes most tools transparently; chain multiple SOCKS proxies for multi-layer pivots, or combine with jump hosts.
- The proxy inherits the SSH host's network reachability, so it reaches exactly what that host can, which is why a well-placed pivot (a bastion, a dual-homed server) is so valuable.

## Tools

- [proxychains-ng](https://github.com/rofl0r/proxychains-ng)
- [OpenSSH -D](https://man.openbsd.org/ssh)

## References

- [HackTricks: SOCKS and dynamic forwarding](https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding)
