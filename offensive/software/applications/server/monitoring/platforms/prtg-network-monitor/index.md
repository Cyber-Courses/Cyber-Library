---
title: "PRTG Network Monitor: attacking the monitoring server"
order: 3
description: "PRTG is a Windows monitoring platform with a web interface and a default prtgadmin account. Its signature weakness is the notification feature: an authenticated administrator configures a notification that executes a program or script, and a crafted command parameter runs as the PRTG service account, which is SYSTEM, giving remote code execution from a web login."
keywords:
  - prtg
  - prtgadmin
  - notification
  - command injection
  - windows monitoring
---

# PRTG Network Monitor

PRTG Network Monitor is a widely used Windows-based monitoring platform with a web interface (on 80/443, and 8080/8443 for the core). It is attacked primarily through its web authentication and its notification feature. The default administrator is `prtgadmin`/`prtgadmin`, frequently unchanged. Once authenticated as an administrator, PRTG's notifications, which can run an external program or script when an alert fires, are abusable: a crafted command parameter in a notification executes on the server, and because the PRTG core service runs as Local System, that execution is as SYSTEM. So a web login (default or weak credentials) chains directly to SYSTEM-level remote code execution on the monitoring server, which also holds the credentials PRTG uses to monitor the estate.

```bash
curl -sk https://<target>/index.htm | grep -i prtg          # fingerprint
curl -sk https://<target>:8080/public/checklogin.htm --data 'loginurl=&username=prtgadmin&password=prtgadmin'
```

## Subtopics

- **[Enumeration](enumeration.md)**: product and version fingerprinting.
- **[Authentication](authentication.md)**: the default prtgadmin and weak credentials.
- **[Notification command abuse](notification-command-abuse.md)**: SYSTEM RCE via notifications.
- **[Known exploits](known-exploits.md)**: the known PRTG vulnerabilities.

## References

- [PRTG documentation](https://www.paessler.com/manuals/prtg)
- [HackTricks: PRTG](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/prtg)
