---
title: "Phishing par device code : capturer des jetons via le flux device-code"
order: 3
description: "Hameçonner le flux OAuth device-code d'Entra : délivrer un code utilisateur pour capturer les jetons d'accès et de rafraîchissement d'une victime avec TokenTactics."
keywords:
  - phishing par device code
  - OAuth
  - TokenTactics
  - jeton de rafraîchissement
  - flux device code
---

# Phishing par device code

Le **flux device-code** OAuth est conçu pour les appareils à saisie limitée : un client demande un code, l'utilisateur le saisit sur `microsoft.com/devicelogin` et s'authentifie, puis le client interroge pour obtenir les jetons résultants. Hameçonné, il est puissant : vous démarrez le flux, envoyez à la victime le court code utilisateur dans un message crédible, et lorsqu'elle termine la connexion (MFA incluse), vous recevez ses **jetons d'accès et de rafraîchissement**.

## Exécuter le flux

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

## Notes d'exploitation

- La victime effectue une vraie MFA, de sorte que les jetons résultants sont pleinement authentifiés ; le jeton de rafraîchissement livre ensuite des jetons pour d'autres portées first-party.
- Le message fonctionne parce que le domaine (`microsoft.com/devicelogin`) est légitime, ce qui déjoue l'inspection d'URL.
- Les jetons de rafraîchissement d'un client public first-party peuvent souvent être échangés entre ressources (Graph, Azure management, etc.).

## Outils

- **TokenTactics** (`Get-AzureToken`) : demande de device code et capture de jetons.
- **TeamFiltration** : module de phishing par device code.

## Références

- [TokenTactics (rvrsh3ll)](https://github.com/rvrsh3ll/TokenTactics)
- [dirkjanm.io : device code research](https://dirkjanm.io/)
- [HackTricks Cloud : device code phishing](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
