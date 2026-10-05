---
title: "Exposed web interface: Zabbix reachable from untrusted networks"
description: "The Zabbix frontend and API are often published to the internet or a flat internal network, frequently over plain HTTP. Exposure makes the version, guest access, default credentials, and any frontend vulnerability reachable by anyone, and a plain-HTTP frontend additionally leaks session cookies, turning a management console into an internet-facing attack surface."
keywords:
  - exposed zabbix
  - internet-facing
  - plain http
  - frontend
  - attack surface
---

# Exposed web interface

Zabbix's frontend (`/zabbix/`) and API (`/api_jsonrpc.php`) are management surfaces that should be tightly restricted, but they are routinely reachable from untrusted networks, published to the internet for remote administration, or open on a flat internal network, and often served over plain HTTP. That exposure puts every other Zabbix weakness within reach of anyone who can connect: the unauthenticated version disclosure, the default `Admin`/`zabbix` and guest access, and any frontend or SQLi vulnerability. A plain-HTTP frontend compounds it by exposing the `zbx_sessionid` cookie to anyone on the path, so an administrator's session is captured in transit. An exposed Zabbix is thus both a direct target and a credential-leak point.

```bash
# discover exposed Zabbix
nmap -p80,443,8080 --open <range>
curl -sk https://<target>/zabbix/ | grep -i zabbix
# search engines index exposed consoles (external): "Zabbix" login pages on 80/443/10051
# confirm plain HTTP (session cookie exposed in transit)
curl -sI http://<target>/zabbix/index.php | grep -i location
```

## Exploitation notes

- Exposure is the force multiplier: it makes [default credentials](../authentication/default-credentials.md), [guest access](../authentication/guest-access.md), version disclosure, and [frontend exploits](../code-execution/known-server-exploits.md) reachable by any attacker, not just an insider.
- A plain-HTTP frontend leaks the session cookie to a sniffer, enabling [session theft](../authentication/session-and-token-theft.md) of an administrator in transit; treat an HTTP Zabbix as credential-exposing.
- The trapper (10051) and agents (10050) are separate exposed surfaces; an internet-reachable trapper or broadly-reachable agents add the [server-exploit](../code-execution/known-server-exploits.md) and [agent-command](../code-execution/agent-remote-commands.md) surfaces.
- Internet-facing Zabbix is a known initial-access target; discovery plus default/guest access is frequently the whole chain.

## References

- [Zabbix: securing the frontend](https://www.zabbix.com/documentation/current/en/manual/installation/requirements/best_practices)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
