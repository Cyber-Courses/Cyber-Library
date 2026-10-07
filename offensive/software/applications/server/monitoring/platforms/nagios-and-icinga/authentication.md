---
title: "Authentication: default and weak Nagios/Icinga credentials"
order: 1
description: "The Nagios and Icinga web interfaces authenticate with a password, and the defaults and weak choices are the way in: nagiosadmin with an install-time password on Core, admin or nagiosadmin on Nagios XI, and weak or reused passwords generally. A web login grants access to the monitoring data and, on XI, to the features and vulnerabilities that lead to code execution."
keywords:
  - nagiosadmin
  - admin
  - default credentials
  - weak password
  - nagios login
---

# Authentication

The web interfaces gate access with a password, and defaults and weak credentials are the common entry. Nagios Core's CGI interface uses an htpasswd account, conventionally `nagiosadmin`, with a password set at install that is frequently weak or left at a documented value. Nagios XI ships an admin account (`admin` or `nagiosadmin`) with a default or install-time password, and its web login is the gateway to the whole application. Icinga Web 2 similarly has an admin account. Weak and reused passwords apply across all three. A successful login grants the monitoring view (host inventory, a map of the estate) and, on Nagios XI especially, access to the configuration and administrative features that the code-execution vulnerabilities build on.

```bash
# Nagios Core CGI (HTTP basic against the nagios cgi)
curl -sk -u nagiosadmin:nagiosadmin https://<target>/nagios/cgi-bin/status.cgi
# Nagios XI login (form) and API
curl -sk -c cj https://<target>/nagiosxi/login.php --data 'username=admin&password=admin&loginButton='
# spray weak/default passwords against the identified product's login
```

## Exploitation notes

- Try the product's documented defaults first: `nagiosadmin` on Core, `admin`/`nagiosadmin` on XI, and the install-time defaults; these are frequently unchanged.
- Nagios XI access is the higher-value login because the application exposes the features and endpoints behind the [XI exploit chains](nagios-xi-exploits.md) and [command injection](command-and-plugin-injection.md); Core's CGI login mainly yields the monitoring view.
- Weak/reused passwords and no lockout on older setups make spraying effective; derive candidates from the organization.
- A web login routes to [command injection](command-and-plugin-injection.md) and, on XI, to the authenticated portions of the [exploit chains](nagios-xi-exploits.md).

## References

- [Nagios XI administration](https://www.nagios.com/products/nagios-xi/)
- [HackTricks: Nagios](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
