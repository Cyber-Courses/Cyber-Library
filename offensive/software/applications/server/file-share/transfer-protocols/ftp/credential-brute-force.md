---
title: "Credential brute force: guessing FTP logins"
description: "Brute-forcing FTP credentials against the cleartext login on port 21, using common and default username and password combinations, since FTP has no lockout by default and transmits credentials in the clear."
keywords:
  - FTP brute force
  - hydra
  - medusa
  - weak passwords
  - cleartext
---

# Credential brute force

FTP authenticates over a cleartext channel with no built-in lockout, so it is a straightforward brute-force and spray target. Vendor defaults and weak user passwords are common, and because the channel is unencrypted, credentials are also recoverable by sniffing an existing session.

```bash
hydra -L users.txt -P passwords.txt ftp://<target>
medusa -h <target> -U users.txt -P passwords.txt -M ftp
# Captured FTP traffic reveals USER/PASS in cleartext
```

## Exploitation notes

- No default lockout means aggressive brute force is viable, but still rate-limit to avoid tripping monitoring.
- Try device and application defaults first; appliances and embedded FTP servers ship with known credentials.
- Sniffing a legitimate FTP session recovers the credentials without guessing, since USER and PASS are plaintext.

## References

- [HackTricks: pentesting FTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ftp/index.html)
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)
