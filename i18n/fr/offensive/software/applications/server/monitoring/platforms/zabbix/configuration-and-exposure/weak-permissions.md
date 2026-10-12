---
title: "Permissions faibles : rôles et groupes d'utilisateurs Zabbix surprivilégiés"
description: "Zabbix contrôle l'accès via des rôles et des permissions de groupes d'utilisateurs, et celles-ci sont fréquemment trop larges : des comptes ordinaires se voient accorder le rôle de Super admin, ou des groupes d'utilisateurs disposent d'un accès en écriture aux hôtes et aux fonctionnalités de script/item. Les comptes surprivilégiés transforment un identifiant de faible valeur en configuration complète et exécution de code, de sorte que le privilège effectif de tout compte compromis doit être vérifié."
keywords:
  - weak permissions
  - super admin
  - user group
  - role
  - privilege
---

# Permissions faibles

Zabbix autorise les actions via des rôles d'utilisateur (quelles fonctionnalités de l'interface/API un utilisateur peut utiliser) et des permissions de groupes d'utilisateurs (quels groupes d'hôtes un utilisateur peut lire ou écrire). Celles-ci sont couramment configurées de façon trop large : des comptes de service et d'opérateur se voient attribuer le rôle de Super admin, ou des groupes d'utilisateurs se voient accorder un accès en écriture aux hôtes et la permission de créer des items et d'exécuter des scripts, bien au-delà de ce que leur usage requiert. La conséquence pour un attaquant est un effet de levier : compromettre même un compte de faible valeur peut rapporter une configuration complète et l'exécution de code si ce compte était surprivilégié. Il faut donc vérifier, plutôt que supposer, le privilège effectif de tout compte obtenu, et les comptes non-admin surpermissionnés sont une voie discrète vers les mêmes résultats que l'admin.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# what can this account actually do? (role features + host-group permissions)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"role.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"usergroup.get","params":{"output":"extend","selectRights":"extend"},"auth":"'"$TOK"'","id":1}'
# test the high-value capabilities directly: can you create a script or an item?
```

## Notes d'exploitation

- Vérifiez le privilège effectif, pas le nom du compte : un compte d'opérateur ou de service surprivilégié peut disposer de droits de Super admin ou de création de script/item, ce qui équivaut à l'admin pour atteindre l'[exécution de code](../code-execution/index.md).
- Les capacités clés à tester sont la gestion de scripts (RCE serveur via les [scripts](../code-execution/global-and-alert-scripts.md)) et l'écriture d'items sur les hôtes (RCE agent via les [items](../code-execution/item-command-execution.md)) ; `role.get`/`usergroup.get` révèlent si le compte les détient.
- Un accès en écriture trop large aux groupes d'hôtes permet aussi à un attaquant d'ajouter des hôtes et des interfaces pointant vers des systèmes qu'il contrôle, ou de reconfigurer la supervision pour récolter davantage d'identifiants.
- Cette faiblesse rend les identifiants de moindre valeur (plus faciles à obtenir) aussi utiles que celui de l'admin ; priorisez l'énumération des droits réels de chaque compte obtenu.

## Références

- [Zabbix : user roles](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/user_roles)
- [Zabbix : permissions](https://www.zabbix.com/documentation/current/en/manual/config/users_and_usergroups/permissions)
