---
order: 4
title: "Mots de passe faibles : identifiants Zabbix courants et réutilisés"
description: "Là où le mot de passe Admin est modifié, Zabbix n'applique aucune politique de mot de passe forte par défaut, de sorte que le remplacement est souvent faible ou réutilisé : zabbix123, le nom de l'entreprise, un motif saison-et-année, ou un mot de passe réutilisé d'ailleurs. Combiné à la liste d'utilisateurs énumérée, une courte pulvérisation ciblée récupère fréquemment un compte fonctionnel."
keywords:
  - weak password
  - zabbix123
  - password policy
  - reuse
  - spray
---

# Mots de passe faibles

Les anciennes versions de Zabbix n'imposent aucune politique de mot de passe, de sorte que même lorsque la valeur par défaut est modifiée, le remplacement est fréquemment faible : `zabbix`, `zabbix123`, `Password1`, le nom de l'entreprise ou du système, ou un motif saison-et-année, et les administrateurs réutilisent couramment un mot de passe employé ailleurs. Combiné à la [liste d'utilisateurs énumérée](../enumeration/user.md), cela rend efficace une pulvérisation de mot de passe courte et ciblée : un mot de passe probable sur les comptes connus, en se concentrant sur les utilisateurs Super admin. La connexion web et l'API valident toutes deux les identifiants, de sorte que l'attaque s'exécute contre l'une ou l'autre.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# spray one likely password across enumerated users via the API
for u in Admin zabbix monitoring operator; do
  r=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"'"$u"'","password":"zabbix123"},"id":1}')
  echo "$u: $r"; done
```

## Notes d'exploitation

- Pulvérisez un ou quelques mots de passe dérivés du contexte sur les utilisateurs énumérés plutôt qu'un brute force profond ; ciblez d'abord les comptes Super admin, car leur accès déverrouille l'exécution de code.
- Les versions plus récentes de Zabbix ont ajouté une politique de mot de passe configurable (et un verrouillage), de sorte que la faisabilité dépend de la [version](../enumeration/version-detection.md) ; les anciennes instances ne sont pas protégées.
- La réutilisation d'identifiants est courante : un mot de passe trouvé ailleurs dans l'environnement peut être celui de l'admin Zabbix, et le mot de passe Zabbix peut être réutilisé pour d'autres services.
- La valeur d'un compte récupéré dépend de son rôle ; confirmez avec `user.get`/`role.get` et dirigez-vous vers l'[exécution de code](../code-execution/index.md) s'il est admin.

## Références

- [Zabbix: password policy](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/authentication)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
