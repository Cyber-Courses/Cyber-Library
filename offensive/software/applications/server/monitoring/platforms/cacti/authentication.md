---
title: "Authentication: the default admin and weak Cacti credentials"
description: "Cacti installs with a default admin account (admin/admin), and although newer versions force a password change on first login, many instances retain weak or guessable passwords. A web login grants access to the data sources and settings that hold device credentials and to the authenticated vulnerability paths that reach code execution."
keywords:
  - cacti admin
  - default credentials
  - weak password
  - first login
  - cacti
---

# Authentication

Cacti's default administrator is `admin` with the password `admin`. Newer versions prompt to change it on first login, but installations that were upgraded, scripted, or simply ignored frequently retain `admin`/`admin` or a weak replacement, and Cacti has historically applied no strong password policy. A web login as admin grants access to the data-source and device configuration, where the SNMP strings and credentials Cacti uses to poll are stored, and to the authenticated command-injection and configuration features that several of its vulnerabilities exploit. So default or weak credentials are the common access step before credential harvesting or an authenticated RCE chain.

```bash
# default admin login (form posts to index.php with a CSRF token)
curl -sk -c cj https://<target>/cacti/index.php | grep -oE 'csrfMagicToken="[^"]+"'   # grab token
curl -sk -b cj -c cj https://<target>/cacti/index.php \
  --data 'action=login&login_username=admin&login_password=admin&__csrf_magic=<token>'
```

## Exploitation notes

- Try `admin`/`admin` first; despite the first-login prompt, it persists on many instances, and weak replacements are common.
- Cacti's login uses a CSRF token (`__csrf_magic`) that must be fetched and submitted; scripted logins grab it from the login page first.
- Admin access unlocks [credential harvesting](credential-harvesting.md) (the stored SNMP/device credentials) and the authenticated portions of the [exploit chains](known-exploits.md); some Cacti RCE is unauthenticated and needs no login.
- Weak/reused passwords and no lockout on older versions make spraying viable.

## References

- [Cacti: user management](https://docs.cacti.net/)
- [HackTricks: Cacti](https://book.hacktricks.xyz/)
