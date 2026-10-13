---
title: "Interface web exposée : Zabbix accessible depuis des réseaux non fiables"
description: "Le frontend et l'API de Zabbix sont souvent publiés sur Internet ou sur un réseau interne plat, fréquemment en HTTP en clair. L'exposition rend la version, l'accès invité, les identifiants par défaut et toute vulnérabilité du frontend accessibles à n'importe qui, et un frontend en HTTP en clair divulgue en outre les cookies de session, transformant une console d'administration en surface d'attaque exposée sur Internet."
keywords:
  - exposed zabbix
  - internet-facing
  - plain http
  - frontend
  - attack surface
---

# Interface web exposée

Le frontend (`/zabbix/`) et l'API (`/api_jsonrpc.php`) de Zabbix sont des surfaces d'administration qui devraient être strictement restreintes, mais elles sont couramment accessibles depuis des réseaux non fiables, publiées sur Internet pour l'administration à distance, ou ouvertes sur un réseau interne plat, et souvent servies en HTTP en clair. Cette exposition met toutes les autres faiblesses de Zabbix à portée de quiconque peut se connecter : la divulgation de version non authentifiée, l'accès `Admin`/`zabbix` par défaut et l'accès invité, et toute vulnérabilité de frontend ou SQLi. Un frontend en HTTP en clair aggrave la situation en exposant le cookie `zbx_sessionid` à quiconque se trouve sur le chemin, de sorte que la session d'un administrateur est capturée en transit. Un Zabbix exposé est donc à la fois une cible directe et un point de fuite d'identifiants.

```bash
# discover exposed Zabbix
nmap -p80,443,8080 --open <range>
curl -sk https://<target>/zabbix/ | grep -i zabbix
# search engines index exposed consoles (external): "Zabbix" login pages on 80/443/10051
# confirm plain HTTP (session cookie exposed in transit)
curl -sI http://<target>/zabbix/index.php | grep -i location
```

## Notes d'exploitation

- L'exposition est le multiplicateur de force : elle rend les [identifiants par défaut](../authentication/default-credentials.md), l'[accès invité](../authentication/guest-access.md), la divulgation de version et les [exploits de frontend](../code-execution/known-server-exploits.md) accessibles à tout attaquant, pas seulement à un initié.
- Un frontend en HTTP en clair divulgue le cookie de session à un renifleur, permettant le [vol de session](../authentication/session-and-token-theft.md) d'un administrateur en transit ; traitez un Zabbix en HTTP comme exposant des identifiants.
- Le trapper (10051) et les agents (10050) sont des surfaces exposées distinctes ; un trapper accessible depuis Internet ou des agents largement accessibles ajoutent les surfaces d'[exploit serveur](../code-execution/known-server-exploits.md) et de [commande d'agent](../code-execution/agent-remote-commands.md).
- Un Zabbix exposé sur Internet est une cible d'accès initial connue ; la découverte combinée à l'accès par défaut/invité constitue fréquemment toute la chaîne.

## Références

- [Zabbix : securing the frontend](https://www.zabbix.com/documentation/current/en/manual/installation/requirements/best_practices)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
