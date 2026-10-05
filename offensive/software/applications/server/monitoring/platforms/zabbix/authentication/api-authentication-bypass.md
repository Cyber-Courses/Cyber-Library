---
title: "API authentication bypass: version-specific Zabbix auth flaws"
description: "Specific Zabbix versions have had authentication-bypass and session-handling flaws in the frontend and API that let an attacker reach authenticated functionality, or act with elevated rights, without valid credentials. These include JSON-RPC and SAML/SSO handling bugs; matching the version to the flaw gives access that chains into the platform's code-execution features."
keywords:
  - zabbix auth bypass
  - json-rpc
  - saml
  - session
  - version-specific
---

# API authentication bypass

Beyond weak credentials, Zabbix itself has shipped authentication-bypass and session-handling vulnerabilities, so against an affected version an attacker reaches authenticated functionality without valid credentials. The flaw classes include bugs in the JSON-RPC and frontend request handling that exposed actions meant to require login, and weaknesses in the SAML/SSO and session validation logic that let a request be treated as an authenticated (sometimes privileged) user. The impact is access to the API/UI, which then chains into the platform's own code-execution features (scripts, items) or into direct exploitation. These are version-specific, so fingerprinting the build and matching the advisory is the method.

```bash
# fingerprint first (the applicable bypass is version-specific)
curl -sk https://<target>/zabbix/api_jsonrpc.php -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
# the bypass itself is specific to the version/flaw (e.g. crafted JSON-RPC calls that
# reach protected methods, or SAML/session handling that yields an authenticated session);
# match the Zabbix build to the advisory for the exact request.
```

## Exploitation notes

- The entry is version-specific: `apiinfo.version` gives the build unauthenticated, which maps to the applicable auth-bypass or session flaw; there is no single universal bypass.
- SAML/SSO handling bugs are notable because they can yield an authenticated, sometimes admin, session; where SSO is configured, assess it against the version.
- A bypass that reaches the API is as good as a login for what follows: enumerate hosts/macros and drive [code execution](../code-execution/index.md) via scripts or items.
- Where no bypass applies, fall back to credentials ([default](default-credentials.md)/[weak](weak-passwords.md)) and [session theft](session-and-token-theft.md).

## References

- [Zabbix security advisories](https://support.zabbix.com/browse/ZBX)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
