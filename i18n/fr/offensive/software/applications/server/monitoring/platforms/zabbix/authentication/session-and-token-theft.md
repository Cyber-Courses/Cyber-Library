---
order: 6
title: "Vol de session et de jeton : rejouer les sessions Zabbix volées"
description: "Une session Zabbix est un cookie zbx_sessionid, et les versions plus récentes ajoutent des jetons d'API à longue durée de vie ; l'un comme l'autre authentifie sans mot de passe. Volé dans un navigateur, capturé en transit sur du HTTP en clair, ou lu dans la table sessions de la base de données Zabbix, une session ou un jeton est rejoué directement pour endosser l'accès de l'utilisateur, contournant entièrement la connexion."
keywords:
  - zbx_sessionid
  - api token
  - session theft
  - cookie
  - zabbix database
---

# Vol de session et de jeton

L'état d'authentification dans Zabbix est un identifiant de type bearer, de sorte que le voler contourne la connexion. La session web est un cookie `zbx_sessionid` ; le présenter revient à être cet utilisateur pendant la durée de vie de la session. Les versions plus récentes de Zabbix ajoutent des jetons d'API, qui sont des chaînes à longue durée de vie créées par utilisateur et envoyées comme identifiant bearer, pratiques pour l'automatisation et durables s'ils sont volés. L'un comme l'autre s'obtient de plusieurs façons : prélevé dans le navigateur d'un administrateur connecté (XSS, accès local), capturé en transit là où le frontend fonctionne en HTTP en clair, ou lu directement dans la table `sessions` de la base de données Zabbix (et les jetons dans la table `token`) si l'attaquant atteint la base. Une session ou un jeton volé est rejoué directement vers l'API ou l'UI pour endosser l'accès de son propriétaire sans aucun mot de passe.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# replay a stolen web session cookie
curl -sk https://<target>/zabbix/zabbix.php?action=dashboard.view -b 'zbx_sessionid=<stolen>'
# replay a stolen API token (bearer) on the API
curl -sk $Z -H "$H" -H 'Authorization: Bearer <stolen-token>' \
  -d '{"jsonrpc":"2.0","method":"host.get","params":{},"id":1}'
# from DB access: sessions and tokens are stored in the Zabbix schema
#   SELECT sessionid, userid FROM sessions;   SELECT token, userid FROM token;
```

## Notes d'exploitation

- Le cookie de session et le jeton d'API sont des identifiants bearer : aucun mot de passe n'est nécessaire pour les rejouer, de sorte que l'attaque est capturer-puis-réutiliser dans la validité de l'identifiant.
- Les frontends Zabbix en HTTP en clair laissent fuiter le `zbx_sessionid` à un sniffer ; une session admin capturée ainsi est un accès admin immédiat, de sorte qu'un frontend non chiffré est une exposition directe.
- L'accès à la base de données est une source puissante : les tables `sessions` et `token` détiennent les sessions et jetons actifs, de sorte qu'une injection SQL ou une compromission de la base livre une authentification réutilisable (et l'injection SQL est elle-même une surface Zabbix, voir [Injection SQL vers RCE](../code-execution/sql-injection-to-rce.md)).
- Les jetons d'API sont à longue durée de vie et survivent aux changements de mot de passe jusqu'à leur révocation, ce qui rend un jeton volé durable ; préférez-le pour la persistance. Une session admin rejouée mène à l'[exécution de code](../code-execution/index.md).

## Références

- [Zabbix: API tokens](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/api_tokens)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
