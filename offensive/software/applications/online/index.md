---
title: "Online: attacking vendor-hosted infrastructure and services"
description: "Attacking services a third party operates: cloud provider platforms (IaaS) and SaaS applications, reached through identity, OAuth, API, and tenant configuration rather than by exploiting a binary you can reach. The attack model is the control-plane and the identity graph, not host exploitation."
keywords:
  - cloud security
  - SaaS security
  - IAM
  - OAuth
  - tenant configuration
  - control plane
---

# Online

Online services run on infrastructure a vendor operates, so there is no server binary to reach and exploit. What you attack instead is the account: the identities that hold permissions, the credentials and tokens that stand in for them, the OAuth and federation that grant them, and the tenant configuration that governs them. This is the same control-plane, identity-first model whether the service is infrastructure (IaaS) or a finished application (SaaS), which is why it sits apart from the self-hosted services under Server.

## Where a target belongs

```bash
# Self-hosted (Server): a service you can reach as a process and exploit its software
nmap -sV <target>                 # a banner/version you could match to a CVE => Server

# Vendor-hosted (Online): a provider or SaaS endpoint, attacked through identity and API
dig +short <target>               # resolves into a provider range (AWS/Azure/GCP) or a SaaS CNAME
curl -sI https://<target>/        # vendor edge headers, a tenant login, an OAuth/SSO redirect
```

A reachable service binary you could exploit is a [Server](../server/index.md) target. An account reached through a provider API, an OAuth consent, or a SaaS tenant is an Online target.

## Subtopics

- **[Cloud](cloud/index.md)**: the cloud provider platforms (AWS, Azure, GCP), attacked through the control-plane API and the IAM graph.
- **[Identity](identity/index.md)**: vendor-hosted identity providers and directories (Microsoft Entra ID), attacked through sign-in, tokens, OAuth consent, directory roles, and cross-tenant access.

## References

- [HackTricks Cloud](https://cloud.hacktricks.wiki/en/index.html)
- [MITRE ATT&CK: Cloud matrix](https://attack.mitre.org/matrices/enterprise/cloud/)
