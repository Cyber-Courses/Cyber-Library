---
title: "Utilisateur : énumérer les comptes Zabbix"
description: "La méthode d'API user.get et la page d'administration Users listent les comptes Zabbix, leurs noms d'utilisateur, leurs rôles et leurs groupes d'utilisateurs. La liste identifie les comptes administratifs à cibler, confirme si des comptes par défaut comme Admin et guest existent, et alimente le password spraying, puisque les comptes Zabbix sont une voie directe vers les fonctionnalités d'exécution de code de la plateforme."
keywords:
  - zabbix users
  - user.get
  - admin account
  - guest
  - role
---

# Utilisateur

Énumérer les utilisateurs Zabbix identifie qui attaquer et quelle est la valeur de chaque compte. La méthode d'API `user.get` (et la page Administration > Users) renvoie les comptes avec leurs noms d'utilisateur, les rôles attribués et l'appartenance aux groupes d'utilisateurs. Cela confirme si le superadmin `Admin` par défaut et le compte `guest` existent, identifie les comptes administratifs (rôle Super admin) qui déverrouillent l'exécution de code, et produit la liste de noms d'utilisateur pour le password spraying. Comme un compte Zabbix suffisamment privilégié mène directement à l'exécution de commandes sur le serveur et les hôtes supervisés, savoir quels comptes détiennent quels rôles concentre l'attaque sur les identifiants sur les cibles de plus grande valeur.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.get","params":{"output":["userid","username","roleid"],"selectUsrgrps":["name"]},"auth":"'"$TOK"'","id":1}'
# map roles to privileges (which accounts are Super admin)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"role.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'
```

## Notes d'exploitation

- Le mappage rôle/groupe d'utilisateurs est l'essentiel : ciblez les comptes Super admin pour les attaques sur les identifiants, car ils déverrouillent l'[exécution de scripts](../code-execution/global-and-alert-scripts.md) et la configuration complète ; un compte non-admin peut tout de même permettre une exécution fondée sur les items selon les permissions.
- Confirmez les comptes par défaut : un `Admin` présent (mot de passe par défaut `zabbix`) ou un `guest` activé est un gain immédiat via les [identifiants par défaut](../authentication/default-credentials.md)/l'[accès invité](../authentication/guest-access.md).
- La liste de noms d'utilisateur alimente le [brute force](../authentication/brute-force.md) et le spraying ; les connexions Zabbix sont basées sur le web, donc respectez tout verrouillage et préférez le spraying.
- `user.get` nécessite une session authentifiée, donc il suit l'accès initial ; avant l'accès, appuyez-vous sur la connaissance des comptes par défaut et sur la page de connexion.

## Références

- [Zabbix API : user.get](https://www.zabbix.com/documentation/current/en/manual/api/reference/user/get)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
