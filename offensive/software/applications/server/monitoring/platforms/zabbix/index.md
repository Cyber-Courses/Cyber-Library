---
title: "Zabbix: attacking the monitoring platform"
description: "Zabbix is a web-and-agent monitoring platform: a PHP frontend and an API, a server on 10051, and agents on 10050. It is a high-value target because it stores credentials for monitored systems and executes commands by design. The surface is enumeration through the API, authentication (defaults, weak passwords, session theft), code execution through items, scripts, the agent, and SQLi, and configuration abuse."
keywords:
  - zabbix
  - api_jsonrpc.php
  - zabbix agent
  - system.run
  - monitoring
---

# Zabbix

Zabbix is a widely deployed monitoring platform with several components: a PHP web frontend (usually at `/zabbix/`), a JSON-RPC API (`/api_jsonrpc.php`), the Zabbix server (trapper on TCP 10051), and Zabbix agents on monitored hosts (TCP 10050). It is a high-value target for two reasons: it stores credentials to reach the systems it monitors (SNMP strings, SSH and IPMI logins, database and API secrets in item and macro configuration), and it executes commands on hosts as a core feature (items, scripts, remote commands). The attack surface follows: enumerating the environment through the API and UI, gaining access through default and weak credentials or stolen sessions, reaching code execution through items, global/alert scripts, the agent, and SQL injection, and abusing configuration and exposure.

```bash
# fingerprint and reach the components
curl -sk https://<target>/zabbix/ | grep -i zabbix
curl -sk https://<target>/zabbix/api_jsonrpc.php -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'   # version, no auth
nmap -p10050,10051 -sV <target>                 # agent and server
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: version, API, users, and monitored hosts.
- **[Authentication](authentication/index.md)**: defaults, weak passwords, guest, API bypass, session theft.
- **[Code execution](code-execution/index.md)**: items, scripts, the agent, SQLi, and known exploits.
- **[Configuration and exposure](configuration-and-exposure/index.md)**: exposed interface and roles.

## References

- [Zabbix API documentation](https://www.zabbix.com/documentation/current/en/manual/api)
- [Zabbix agent documentation](https://www.zabbix.com/documentation/current/en/manual/concepts/agent)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
