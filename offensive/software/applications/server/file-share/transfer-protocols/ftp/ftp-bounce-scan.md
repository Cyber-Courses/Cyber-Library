---
title: "FTP bounce scan: proxying port scans through the server"
description: "The FTP PORT command lets a client specify the address for the data connection, and a server that does not restrict it to the client's address can be told to open data connections to arbitrary hosts and ports. This bounce turns the FTP server into a scan proxy, reaching internal hosts and ports the attacker cannot reach directly."
keywords:
  - ftp bounce
  - port command
  - scan proxy
  - internal network
  - nmap -b
---

# FTP bounce scan

The FTP data connection in active mode is set up with the `PORT` command, where the client tells the server which IP and port to connect to for the transfer. The protocol does not require that address to be the client's own, so a server that fails to restrict `PORT` to the control-connection's peer can be instructed to open data connections to any host and port. By observing whether the data connection succeeds, an attacker uses the FTP server as a proxy to scan hosts and ports it can reach but the attacker cannot, typically internal systems behind the FTP server.

```bash
# nmap's built-in FTP bounce scan: scan <target> via the FTP relay
nmap -b anonymous:a@<ftp-relay> -p 1-1024 <internal-target>
# the relay (ftp-relay) issues PORT to <internal-target>; open vs refused is inferred
# manual: PORT a,b,c,d,p1,p2 then LIST; the server's response reveals reachability
```

The scan infers port state from the FTP server's response to a data-transfer command after a crafted `PORT`: a successful data connection implies the target port is open, a refusal implies closed or filtered. This reaches the internal network from the FTP server's vantage point.

## Exploitation notes

- The value is pivoting: the FTP server scans internal hosts and ports unreachable from the attacker, mapping a network segment behind it without any foothold on the server itself.
- It needs a server that does not bind `PORT` targets to the client address; modern servers usually restrict this, so it is mostly found on old or misconfigured FTP daemons.
- Beyond scanning, bounce can in some cases deliver bytes to an internal service (a limited data-write proxy), though port mapping is the common use.
- `nmap -b` automates it; `anonymous` access to the relay is enough when anonymous is allowed.

## References

- [nmap FTP bounce (-b)](https://nmap.org/book/scan-methods-ftp-bounce-scan.html)
- [RFC 959: PORT command](https://datatracker.ietf.org/doc/html/rfc959)
