---
title: "Authentication: default and weak LibreNMS/Observium credentials"
description: "The LibreNMS and Observium web interfaces authenticate with a password and are exposed to default and weak credentials, including install-time admin accounts and reused passwords. A web login grants access to the device configuration holding stored credentials and to the authenticated features behind the command-injection vulnerabilities."
keywords:
  - librenms login
  - observium
  - default credentials
  - weak password
  - admin
---

# Authentication

Both platforms gate their web interface with a password, and access is usually gained through default or weak credentials. LibreNMS creates an admin account at install whose password is operator-chosen and often weak or reused; Observium ships with a documented default (`admin`/`admin` in its setup) that is frequently unchanged. Neither enforces a strong policy by default on older versions. A web login grants the monitoring view and the device/data-source configuration where the SNMP strings and credentials are stored, and reaches the authenticated features that the command-injection vulnerabilities exploit, so credentials are the access step before both harvesting and RCE.

```bash
# Observium default
curl -sk -c cj https://<target>/ --data 'username=admin&password=admin'
# LibreNMS form login (CSRF token in the login page)
curl -sk -c cj https://<target>/login                       # grab _token
curl -sk -b cj -c cj https://<target>/login --data '_token=<t>&username=admin&password=<pw>'
# spray weak/reused passwords against the identified product
```

## Exploitation notes

- Try the product's defaults (`admin`/`admin` on Observium; the install admin on LibreNMS) and weak/reused passwords; no strong policy on older versions makes spraying effective.
- Admin access unlocks [credential harvesting](credential-harvesting.md) (stored device credentials) and the authenticated [command-injection](command-injection.md) paths to RCE.
- LibreNMS also issues API tokens; a found token authenticates to the API directly, bypassing the form.
- A login is the common precondition for both follow-ons; non-admin accounts have limited reach.

## References

- [LibreNMS: authentication](https://docs.librenms.org/Extensions/Authentication/)
- [Observium documentation](https://docs.observium.org/)
