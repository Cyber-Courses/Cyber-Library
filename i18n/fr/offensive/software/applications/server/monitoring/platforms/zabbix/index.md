---
title: "Zabbix : attaquer la plateforme de supervision"
order: 1
description: "Zabbix est une plateforme de supervision web et agent : un frontend PHP et une API, un serveur sur 10051, et des agents sur 10050. C'est une cible de grande valeur car elle stocke les identifiants des systèmes supervisés et exécute des commandes par conception. La surface est l'énumération via l'API, l'authentification (valeurs par défaut, mots de passe faibles, vol de session), l'exécution de code via les items, les scripts, l'agent et la SQLi, et l'abus de configuration."
keywords:
  - zabbix
  - api_jsonrpc.php
  - zabbix agent
  - system.run
  - monitoring
---

# Zabbix

Zabbix est une plateforme de supervision largement déployée comportant plusieurs composants : un frontend web PHP (généralement sur `/zabbix/`), une API JSON-RPC (`/api_jsonrpc.php`), le serveur Zabbix (trapper sur le port TCP 10051), et des agents Zabbix sur les hôtes supervisés (TCP 10050). C'est une cible de grande valeur pour deux raisons : elle stocke les identifiants pour atteindre les systèmes qu'elle supervise (chaînes SNMP, connexions SSH et IPMI, secrets de base de données et d'API dans la configuration des items et des macros), et elle exécute des commandes sur les hôtes comme fonctionnalité centrale (items, scripts, commandes distantes). La surface d'attaque en découle : énumérer l'environnement via l'API et l'interface, obtenir un accès via des identifiants par défaut et faibles ou des sessions volées, atteindre l'exécution de code via les items, les scripts globaux/d'alerte, l'agent et l'injection SQL, et abuser de la configuration et de l'exposition.

```bash
# fingerprint and reach the components
curl -sk https://<target>/zabbix/ | grep -i zabbix
curl -sk https://<target>/zabbix/api_jsonrpc.php -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'   # version, no auth
nmap -p10050,10051 -sV <target>                 # agent and server
```

## Sous-thèmes

- **[Énumération](enumeration/index.md)** : version, API, utilisateurs et hôtes supervisés.
- **[Authentification](authentication/index.md)** : valeurs par défaut, mots de passe faibles, invité, contournement de l'API, vol de session.
- **[Exécution de code](code-execution/index.md)** : items, scripts, l'agent, SQLi et vulnérabilités connues.
- **[Configuration et exposition](configuration-and-exposure/index.md)** : interface exposée et rôles.

## Références

- [Zabbix API documentation](https://www.zabbix.com/documentation/current/en/manual/api)
- [Zabbix agent documentation](https://www.zabbix.com/documentation/current/en/manual/concepts/agent)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
