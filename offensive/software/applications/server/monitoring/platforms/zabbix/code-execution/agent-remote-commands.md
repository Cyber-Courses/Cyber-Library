---
title: "Agent remote commands: executing against the Zabbix agent"
order: 5
description: "The Zabbix agent on TCP 10050 answers item requests, and where remote commands are permitted it runs system.run directly. An attacker who can reach an agent sends a system.run request with zabbix_get and the command executes on that host as the agent's account, no server or login required, so an exposed, permissively-configured agent is direct code execution."
keywords:
  - zabbix agent
  - 10050
  - zabbix_get
  - system.run
  - enableremotecommands
---

# Agent remote commands

The Zabbix agent (`zabbix_agentd`) listens on TCP 10050 and answers item-key requests from whoever can connect, normally the Zabbix server. Where the agent is configured to allow remote commands, it executes `system.run[command]` directly, so an attacker who can reach the agent runs commands on its host without going through the server or authenticating to the frontend at all. The gate is the agent's configuration: older agents enable this with `EnableRemoteCommands=1`, and newer agents require an explicit `AllowKey=system.run[*]` (it is denied by default in current versions). An agent with either setting, especially one exposed beyond the server's address, is direct code execution on that host.

```bash
# query the agent directly; if system.run is allowed, it runs the command
zabbix_get -s <agent-host> -p 10050 -k 'system.run[id]'
zabbix_get -s <agent-host> -p 10050 -k 'system.run[cat /etc/passwd]'
# without zabbix_get, the agent protocol is simple enough to speak over a socket
printf 'system.run[id]\n' | nc <agent-host> 10050
# first confirm the agent answers and which keys are permitted
zabbix_get -s <agent-host> -p 10050 -k agent.ping
```

## Exploitation notes

- No server or login is needed: reaching 10050 and a permissive agent config (`EnableRemoteCommands` or `AllowKey=system.run[*]`) is sufficient, so a widely-reachable agent is a direct foothold on its host.
- Modern agents deny `system.run` by default, so this depends on configuration; test `system.run[id]` and fall back to reading non-command keys (`agent.ping`, `system.uname`) to confirm reachability and version.
- Agents are often only firewalled to the Zabbix server, so this is most useful from the server's vantage (after compromising the server) to pivot to every monitored host, or where agents are exposed more broadly.
- This is the agent-side counterpart to [item command execution](item-command-execution.md) (which reaches the agent via the server) and complements server-side [scripts](global-and-alert-scripts.md).

## Tools

- [zabbix_get](https://www.zabbix.com/documentation/current/en/manpages/zabbix_get)

## References

- [Zabbix agent: system.run and AllowKey/DenyKey](https://www.zabbix.com/documentation/current/en/manual/config/items/restrict_checks)
- [HackTricks: Zabbix agent](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
