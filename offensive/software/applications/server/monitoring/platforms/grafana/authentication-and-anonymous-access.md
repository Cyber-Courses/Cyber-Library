---
title: "Authentication and anonymous access: reaching Grafana"
order: 1
description: "Grafana ships with admin/admin and, where configured, allows anonymous access that grants a viewer (or higher) role without login. Default credentials, weak passwords, and an over-permissive anonymous organization role each give access to dashboards and, with enough privilege, to the data sources and their stored credentials."
keywords:
  - grafana admin
  - anonymous access
  - default credentials
  - viewer role
  - org role
---

# Authentication and anonymous access

Grafana access comes from credentials or from anonymous access. The default administrator is `admin`/`admin`, prompted to change on first login but frequently left or set weakly. Grafana also supports anonymous access (`[auth.anonymous] enabled = true`) that assigns unauthenticated visitors an organization role, intended as Viewer but sometimes configured as Editor or Admin, which grants access without any login. So reaching Grafana is often as simple as browsing to it (anonymous) or trying the default, and the role obtained determines the impact: a Viewer sees dashboards and the data they reveal, while an Editor/Admin reaches the data sources and the features that expose their stored credentials and the SSRF proxy.

```bash
# default admin
curl -sk https://<target>:3000/login -H 'Content-Type: application/json' \
  -d '{"user":"admin","password":"admin"}'
# anonymous access: dashboards/data reachable with no login if enabled
curl -sk https://<target>:3000/api/search                 # lists dashboards if anon-enabled
curl -sk https://<target>:3000/api/org                     # the anonymous org/role
```

## Exploitation notes

- Try anonymous access first (it needs no credential) and the `admin`/`admin` default; both are common, and an over-permissive anonymous role (Editor/Admin) is immediate elevated access.
- The role decides reach: Viewer exposes dashboards and query results (data disclosure), while Editor/Admin reaches data-source configuration and the proxy, see [Data source SSRF and credentials](data-source-ssrf-and-credentials.md).
- Weak/reused admin passwords and API keys are additional routes; a leaked Grafana API key authenticates to the API directly.
- Where credentials and anonymous access fail, the unauthenticated [path traversal](path-traversal-file-read.md) may still read server secrets on a vulnerable version.

## References

- [Grafana: anonymous authentication](https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/)
- [HackTricks: Grafana](https://book.hacktricks.xyz/network-services-pentesting/3000-pentesting-grafana)
