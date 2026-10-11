---
title: "Authentification : obtenir un accès à Zabbix"
order: 1
description: "L'accès à Zabbix s'obtient via la connexion web ou l'API : les identifiants Admin/zabbix par défaut, les mots de passe faibles et la force brute, un compte invité activé, les failles de contournement d'authentification de l'API, et la réutilisation de cookies de session ou de jetons d'API volés. Chacune de ces voies fournit une session dont le privilège, en particulier Super admin, détermine la voie vers l'exécution de code."
keywords:
  - zabbix authentication
  - default credentials
  - guest
  - api token
  - session
---

# Authentification

Obtenir un accès à Zabbix est la porte d'entrée vers ses fonctionnalités d'exécution de code et ses identifiants stockés, et il existe plusieurs voies. Le compte superadmin `Admin` par défaut est livré avec le mot de passe `zabbix` et est fréquemment laissé inchangé. Les mots de passe faibles et l'absence de politique stricte rendent la force brute et la pulvérisation efficaces. Le compte `guest` intégré, lorsqu'il est activé, accorde un accès en lecture non authentifié. Certaines versions ont connu des failles de contournement d'authentification de l'API. Et les cookies de session ou jetons d'API volés sont rejoués directement. Ce qui importe après l'accès, c'est le rôle du compte : un Super admin déverrouille l'exécution de scripts et la configuration complète, tandis que des rôles moindres peuvent tout de même permettre une exécution basée sur les éléments selon les permissions.

```bash
# web login and API login both validate the same credentials
Z=https://<target>/zabbix/api_jsonrpc.php
curl -sk $Z -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}'
```

## Sous-thèmes

- **[Identifiants par défaut](default-credentials.md)** : Admin/zabbix et invité.
- **[Mots de passe faibles](weak-passwords.md)** : mots de passe courants et réutilisés.
- **[Force brute](brute-force.md)** : attaques en ligne sur la page de connexion.
- **[Accès invité](guest-access.md)** : le compte invité en lecture seule intégré.
- **[Contournement d'authentification de l'API](api-authentication-bypass.md)** : failles de contournement d'authentification propres à la version.
- **[Vol de session et de jeton](session-and-token-theft.md)** : rejeu de sessions et de jetons d'API volés.

## Références

- [Documentation d'authentification Zabbix](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
