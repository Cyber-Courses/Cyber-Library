---
title: "Authentication: the default prtgadmin and weak credentials"
description: "PRTG ships with a default administrator account, prtgadmin/prtgadmin, that is frequently unchanged, and otherwise accepts weak passwords. A web login as an administrator is the whole precondition for the notification-based SYSTEM code execution, so default or sprayed credentials against the PRTG login are the direct path to compromising the server."
keywords:
  - prtgadmin
  - default credentials
  - weak password
  - prtg login
  - administrator
---

# Authentication

PRTG's web interface authenticates with a username and password, and the way in is almost always credentials. The default administrator is `prtgadmin` with the password `prtgadmin`, left unchanged on a large share of installations. Where it is changed, weak or reused passwords are common, and older PRTG versions apply no strong policy. Because administrator access is the sole precondition for the notification-based code execution that runs as SYSTEM, a successful login is effectively server compromise, so trying the default and spraying a few weak passwords against the login is the primary attack.

```bash
# default admin login (core on 8080 in this example)
curl -sk -c cj 'https://<target>:8080/public/checklogin.htm' \
  --data 'loginurl=/&username=prtgadmin&password=prtgadmin'
# a successful login sets session cookies used for the authenticated API/UI
curl -sk -b cj 'https://<target>:8080/api/table.json?content=sensors&columns=objid,device'
```

## Exploitation notes

- Try `prtgadmin`/`prtgadmin` first; it is the canonical default and grants administrator, which is all that is needed for [notification command abuse](notification-command-abuse.md).
- Weak/reused passwords and no lockout on older versions make spraying effective; a single admin credential is SYSTEM-level RCE, so the login is high-value.
- The login sets session cookies used for the authenticated API (`api/table.json`, notification management), which drive the follow-on.
- An administrator session routes directly to [notification command abuse](notification-command-abuse.md); non-admin access is of limited use here.

## References

- [PRTG: user accounts](https://www.paessler.com/manuals/prtg/user_accounts_settings)
- [HackTricks: PRTG](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/prtg)
