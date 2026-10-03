---
title: "Cloud"
description: "Offensive techniques against cloud platforms, organized by provider because each one's identity model, services, APIs, and tooling differ. A cloud account is identities holding permissions over resources, reached through credentials, so attacks revolve around the IAM graph and the control-plane API rather than host exploits."
keywords:
  - cloud security
  - AWS
  - Azure
  - GCP
  - IAM
---

# Cloud

Cloud platforms are attacked through their **control-plane APIs**, not through host exploitation. A cloud account is a set of **identities** that hold **permissions** over **resources**, reached with **credentials**; so the work is enumerating that graph, obtaining a principal, escalating its permissions, and abusing the services it can reach. Because each provider's identity model, service set, and tooling differ sharply, this area is organized **by provider**, and every provider is broken down the same way, by the **surface** under attack rather than by kill-chain phase.

## How each provider is organized

The same nine surfaces recur under every provider, so a technique sits next to the resource or primitive it abuses:

- **Identity** and **Credentials** are the control-plane core: obtain a principal, then escalate it.
- **Compute**, **Storage**, **Serverless**, **Data**, **Networking**, and **Messaging** are the resource surfaces.
- **Logging and detection** is the audit plane, attacked to blind and evade it.

Enumeration, privilege escalation, lateral movement, persistence, and exfiltration are not separate topics: each lives in the surface it works through. Enumeration is folded into every surface (mapping the IAM graph under identity, finding public buckets under storage); privilege escalation lives under identity, exfiltration under storage and data, and so on. The account-wide inventory tooling that spans surfaces is covered as methodology in each provider's index.

## Providers

- **[AWS](aws/index.md)**: Amazon Web Services, the template for the surface breakdown above.

Azure (resource plane) and GCP follow the same nine-surface shape and are built on the same lines.

## Scope and boundaries

- **Entra ID** (Microsoft's cloud directory) is attacked like a directory, so it lives under [Directory](../directory/index.md) next to Active Directory, not here; the Azure area here covers the **resource plane** (subscriptions, VMs, storage, managed identities).
- **Kubernetes and container escapes** are their own area; the cloud pages cover only the managed-service control plane and cross-reference container internals.
- The on-premises to cloud bridge (Entra Connect, AD FS, PRT) lives with [hybrid identity](../directory/active-directory/trusts/entra-hybrid.md).

## References

- [HackTricks Cloud](https://cloud.hacktricks.wiki/en/index.html)
- [MITRE ATT&CK: cloud matrices](https://attack.mitre.org/matrices/enterprise/cloud/)
