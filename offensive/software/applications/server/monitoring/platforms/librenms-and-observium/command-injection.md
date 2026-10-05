---
title: "Command injection: authenticated RCE in LibreNMS and Observium"
description: "LibreNMS and Observium build shell commands from user-supplied input in several features (device addition, SNMP options, services and collectd parameters), and where that input is not sanitized, an authenticated attacker injects commands that run on the monitoring host as the web or poller user. Matching the version to the affected feature gives code execution."
keywords:
  - librenms command injection
  - observium
  - rce
  - snmp options
  - poller
---

# Command injection

Both platforms shell out to external tools (net-snmp binaries, `fping`, `rrdtool`, collector scripts) built from configuration values, and several of those values are attacker-controlled through the authenticated UI/API. Where the input is placed into a command without sanitization, injecting shell metacharacters runs arbitrary commands on the monitoring host. LibreNMS has had command injection through device and service parameters, SNMP option fields, and collectd/graph inputs; Observium through similar device and option handling. The injected command runs as the web or poller service user. The specific affected feature and whether it requires admin or lower privilege are version-specific, so fingerprinting and matching the advisory selects the path.

```bash
# the injectable field is version/feature-specific; the pattern is a config value
# that reaches a shell command. Examples (authenticated):
#   device "overwrite_ip" / SNMP options field:  127.0.0.1 -c public; id
#   service check parameters / collectd inputs carrying `;`, `|`, $(...)
# the injected command runs on the monitoring host (web/poller user). Match the
# LibreNMS/Observium version to the advisory for the exact field.
curl -sk -b cj 'https://<target>/ajax_form.php' --data 'type=...&snmp_options=127.0.0.1;id'
```

## Exploitation notes

- The injectable inputs are configuration values that reach shell commands: SNMP option strings, device addresses/overwrite fields, and service/collector parameters; test these with metacharacters once authenticated.
- Execution is as the web/poller user on the monitoring host, which has access to the database holding every device's [stored credentials](credential-harvesting.md), so RCE cascades to the estate.
- The exact field and required privilege are version-specific; fingerprint ([enumeration](enumeration.md)) and match the advisory.
- Combine with [authentication](authentication.md) for the access step; from execution, harvest the stored credentials and pivot.

## References

- [LibreNMS security advisories](https://github.com/librenms/librenms/security/advisories)
- [HackTricks](https://book.hacktricks.xyz/)
