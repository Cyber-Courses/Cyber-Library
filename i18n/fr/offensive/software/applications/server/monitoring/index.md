---
title: "Monitoring : attaquer l'infrastructure de monitoring et d'observabilité"
order: 6
description: "Les systèmes de monitoring sont une cible de grande valeur car ils détiennent des identifiants pour l'ensemble du parc qu'ils surveillent et s'y étendent. Le domaine se divise entre les protocoles de télémétrie (SNMP, syslog, NetFlow/IPFIX) et les plateformes qui les collectent et les visualisent (Zabbix, Nagios, PRTG, Cacti, LibreNMS, Grafana, Prometheus, SolarWinds), chacun étant une voie vers l'information, les identifiants et l'exécution de code."
keywords:
  - monitoring
  - snmp
  - zabbix
  - grafana
  - observability
---

# Monitoring

L'infrastructure de monitoring et d'observabilité est une cible d'une valeur disproportionnée. Pour faire son travail, elle doit s'étendre dans chaque système qu'elle surveille, elle détient donc des identifiants pour le parc (chaînes SNMP, identifiants d'équipements, secrets de bases de données et de cloud), cartographie le réseau interne, et exécute souvent des commandes sur les hôtes surveillés par conception. Compromettre une plateforme de monitoring produit donc de la reconnaissance, des identifiants stockés, et fréquemment l'exécution de code sur de nombreux systèmes à la fois. Le domaine se divise en deux couches : les **protocoles** de télémétrie que parlent les agents et les collecteurs, et les **plateformes** qui collectent, stockent et visualisent cette télémétrie.

```bash
# find monitoring surfaces on a host or network
nmap -sU -p161,162 <target>                    # SNMP agent / traps
nmap -p443,80,3000,9090,10050,10051,5666,8086 -sV <target>  # web UIs, Zabbix, NRPE, etc.
```

## Sous-thèmes

- **[Protocoles](protocols/index.md)** : SNMP, syslog et NetFlow/IPFIX.
- **[Plateformes](platforms/index.md)** : Zabbix, Nagios/Icinga, PRTG, Cacti, LibreNMS, Grafana, Prometheus, SolarWinds.

## Références

- [HackTricks: network services pentesting](https://book.hacktricks.xyz/network-services-pentesting)
- [MITRE ATT&CK: network sniffing and valid accounts](https://attack.mitre.org/techniques/T1040/)
