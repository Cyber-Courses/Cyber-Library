---
title: "Code execution: running commands through Zabbix"
order: 4
description: "Zabbix runs commands as a feature, so an authenticated attacker reaches code execution several ways: item keys like system.run that execute on an agent, global and alert scripts that run on the Zabbix server, direct agent remote commands on 10050, SQL-injection chains in the frontend, and known server vulnerabilities. The server and agents run these with their service privileges."
keywords:
  - zabbix rce
  - system.run
  - scripts
  - zabbix agent
  - sql injection
---

# Code execution

Zabbix executes commands by design, to collect data and to react to problems, so for an attacker with sufficient access it is a code-execution platform. There are several distinct primitives. Item keys such as `system.run[]` execute on a Zabbix agent when the item is evaluated. Global scripts and alert (action) scripts run commands on the Zabbix server (or a selected agent) when triggered. The Zabbix agent on 10050 directly executes `system.run` where remote commands are permitted, reachable without the server. SQL injection in the frontend extracts sessions and credentials and chains to the above. And the server and frontend have had their own remote-code-execution vulnerabilities. The server and agents run these commands as their service accounts (often `zabbix`, sometimes more), so execution is a foothold on the monitoring host and, through agents, on monitored hosts.

## Subtopics

- **[Item command execution](item-command-execution.md)**: system.run and command items on agents.
- **[Global and alert scripts](global-and-alert-scripts.md)**: scripts that run on the server.
- **[Agent remote commands](agent-remote-commands.md)**: direct execution against the agent on 10050.
- **[SQL injection to RCE](sql-injection-to-rce.md)**: frontend SQLi and its chains.
- **[Known server exploits](known-server-exploits.md)**: the recurring server/frontend RCE.

## References

- [Zabbix: remote commands and scripts](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/alerts/scripts)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
