---
title: "Exécution de code : lancer des commandes via Zabbix"
order: 4
description: "Zabbix exécute des commandes par conception, si bien qu'un attaquant authentifié atteint l'exécution de code de plusieurs façons : des clés d'item comme system.run qui s'exécutent sur un agent, des scripts globaux et d'alerte qui s'exécutent sur le serveur Zabbix, des commandes distantes directes sur l'agent en 10050, des chaînes d'injection SQL dans le frontend, et des vulnérabilités serveur connues. Le serveur et les agents les exécutent avec leurs privilèges de service."
keywords:
  - zabbix rce
  - system.run
  - scripts
  - zabbix agent
  - sql injection
---

# Exécution de code

Zabbix exécute des commandes par conception, pour collecter des données et réagir aux problèmes, de sorte que pour un attaquant disposant d'un accès suffisant, c'est une plateforme d'exécution de code. Il existe plusieurs primitives distinctes. Des clés d'item telles que `system.run[]` s'exécutent sur un agent Zabbix lorsque l'item est évalué. Les scripts globaux et les scripts d'alerte (action) exécutent des commandes sur le serveur Zabbix (ou sur un agent sélectionné) lorsqu'ils se déclenchent. L'agent Zabbix sur le port 10050 exécute directement `system.run` là où les commandes distantes sont autorisées, accessible sans passer par le serveur. L'injection SQL dans le frontend extrait des sessions et des identifiants et s'enchaîne avec ce qui précède. Et le serveur comme le frontend ont connu leurs propres vulnérabilités d'exécution de code à distance. Le serveur et les agents exécutent ces commandes sous leurs comptes de service (souvent `zabbix`, parfois davantage), de sorte que l'exécution constitue un point d'appui sur l'hôte de supervision et, à travers les agents, sur les hôtes supervisés.

## Sous-thèmes

- **[Exécution de commandes par item](item-command-execution.md)** : system.run et items de commande sur les agents.
- **[Scripts globaux et scripts d'alerte](global-and-alert-scripts.md)** : scripts qui s'exécutent sur le serveur.
- **[Commandes distantes d'agent](agent-remote-commands.md)** : exécution directe contre l'agent en 10050.
- **[De l'injection SQL à la RCE](sql-injection-to-rce.md)** : SQLi dans le frontend et ses chaînes.
- **[Vulnérabilités serveur connues](known-server-exploits.md)** : la RCE serveur/frontend récurrente.

## Références

- [Zabbix : remote commands and scripts](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/alerts/scripts)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
