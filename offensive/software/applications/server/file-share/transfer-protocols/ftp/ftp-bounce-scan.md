---
title: "FTP bounce scan: proxying connections through an FTP server"
description: "Abusing the FTP PORT command to make an FTP server open data connections to arbitrary hosts and ports, using it as a proxy to scan or reach internal systems the attacker cannot connect to directly, the classic FTP bounce attack."
keywords:
  - FTP bounce
  - PORT command
  - proxy scan
  - internal pivot
  - nmap -b
---

# FTP bounce scan

The FTP PORT command tells the server which address and port to open the data connection to. Servers that honor an arbitrary PORT target let an attacker use the FTP server as a proxy: directing it to connect to internal hosts and ports reveals, by the server's response, whether each is open, and can reach services the attacker cannot touch directly.

```bash
# Nmap FTP bounce scan: scan an internal target through the FTP relay
nmap -b anonymous:anon@<ftp-server> -p 1-1000 <internal-target>
```

## Exploitation notes

- The value is reaching an internal network segment through an FTP server that straddles it, bypassing the attacker's own routing and filtering.
- Beyond scanning, a bounce can be used to deliver data to an internal service that accepts the FTP data stream.
- Most modern FTP servers restrict PORT to the client's own address, so bounce works on older or permissive servers.

## References

- [Nmap FTP bounce scan](https://nmap.org/book/scan-methods-ftp-bounce-scan.html)
- [HackTricks: pentesting FTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ftp/index.html)
