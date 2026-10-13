---
order: 5
title: "Commandes distantes de l'agent : exécuter contre l'agent Zabbix"
description: "L'agent Zabbix sur le port TCP 10050 répond aux requêtes d'items, et là où les commandes distantes sont autorisées il exécute system.run directement. Un attaquant capable d'atteindre un agent envoie une requête system.run avec zabbix_get et la commande s'exécute sur cet hôte sous le compte de l'agent, sans serveur ni connexion requis, de sorte qu'un agent exposé et configuré de façon permissive est une exécution de code directe."
keywords:
  - zabbix agent
  - 10050
  - zabbix_get
  - system.run
  - enableremotecommands
---

# Commandes distantes de l'agent

L'agent Zabbix (`zabbix_agentd`) écoute sur le port TCP 10050 et répond aux requêtes de clés d'items de quiconque peut se connecter, normalement le serveur Zabbix. Là où l'agent est configuré pour autoriser les commandes distantes, il exécute `system.run[command]` directement, de sorte qu'un attaquant capable d'atteindre l'agent exécute des commandes sur son hôte sans passer par le serveur ni s'authentifier au frontend. Le verrou est la configuration de l'agent : les anciens agents l'activent avec `EnableRemoteCommands=1`, et les nouveaux agents nécessitent un `AllowKey=system.run[*]` explicite (c'est refusé par défaut dans les versions actuelles). Un agent avec l'un ou l'autre réglage, en particulier un agent exposé au-delà de l'adresse du serveur, est une exécution de code directe sur cet hôte.

```bash
# query the agent directly; if system.run is allowed, it runs the command
zabbix_get -s <agent-host> -p 10050 -k 'system.run[id]'
zabbix_get -s <agent-host> -p 10050 -k 'system.run[cat /etc/passwd]'
# without zabbix_get, the agent protocol is simple enough to speak over a socket
printf 'system.run[id]\n' | nc <agent-host> 10050
# first confirm the agent answers and which keys are permitted
zabbix_get -s <agent-host> -p 10050 -k agent.ping
```

## Notes d'exploitation

- Aucun serveur ni connexion n'est nécessaire : atteindre le port 10050 et une configuration d'agent permissive (`EnableRemoteCommands` ou `AllowKey=system.run[*]`) suffit, de sorte qu'un agent largement accessible est un point d'ancrage direct sur son hôte.
- Les agents modernes refusent `system.run` par défaut, de sorte que cela dépend de la configuration ; testez `system.run[id]` et repliez-vous sur la lecture de clés qui ne sont pas des commandes (`agent.ping`, `system.uname`) pour confirmer l'accessibilité et la version.
- Les agents ne sont souvent filtrés que vers le serveur Zabbix, de sorte que ceci est surtout utile depuis le point de vue du serveur (après l'avoir compromis) pour pivoter vers chaque hôte supervisé, ou là où les agents sont exposés plus largement.
- C'est la contrepartie côté agent de l'[exécution de commandes via les items](item-command-execution.md) (qui atteint l'agent via le serveur) et complète les [scripts](global-and-alert-scripts.md) côté serveur.

## Outils

- [zabbix_get](https://www.zabbix.com/documentation/current/en/manpages/zabbix_get)

## Références

- [Zabbix agent: system.run and AllowKey/DenyKey](https://www.zabbix.com/documentation/current/en/manual/config/items/restrict_checks)
- [HackTricks: Zabbix agent](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
