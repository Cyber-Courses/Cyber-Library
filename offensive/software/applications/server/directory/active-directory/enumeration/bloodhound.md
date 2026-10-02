---
title: "BloodHound: collecting and graphing Active Directory attack paths"
description: "Using BloodHound to collect Active Directory objects, sessions, ACLs, and trusts with SharpHound or bloodhound-python, then analyzing the graph to find privilege-escalation paths to high-value targets."
keywords:
  - BloodHound
  - SharpHound
  - bloodhound-python
  - attack path
  - cypher
---

# BloodHound

BloodHound turns the directory into a graph of principals and the relationships between them (group membership, sessions, ACL rights, delegation, trusts) and then answers the one question that matters: what is the shortest path from what I control to what I want. It replaces guessing with a computed route, and it is the standard first analysis step once you can read the domain.

## Collection

Collectors gather the data; the GUI analyzes it. Collect with whatever matches your position:

```bash
# From a non-domain Linux box, over the network with any domain creds
bloodhound-python -u user -p 'Password1' -d example.local -ns <dc-ip> -c All

# From a domain-joined Windows host
SharpHound.exe -c All
# or the PowerShell collector
Invoke-BloodHound -CollectionMethod All
```

`-c All` (or `All,GPOLocalGroup`) gathers objects, group membership, ACLs, trusts, and sessions. **Sessions and local-admin collection** (who is logged on where, who is local admin on what) are the parts that reveal lateral-movement paths, but they require querying each host and are noisier; `DCOnly` is a quiet alternative that pulls everything readable from the DC without touching workstations.

## Analysis

Load the resulting JSON (or connect the collector to the BloodHound CE database), mark what you own as Owned, mark targets as High Value, and run the path queries:

- **Shortest paths to Domain Admins** from owned principals.
- **Pre-built queries**: Kerberoastable users, AS-REP roastable users, unconstrained-delegation systems, principals with DCSync rights, dangerous ACLs.
- **Custom Cypher** for specific hunts, for example principals with `GenericAll` over a privileged group:

```cypher
MATCH p=(n)-[:GenericAll]->(g:Group) WHERE g.highvalue=true RETURN p
```

Every edge BloodHound draws corresponds to a concrete technique in the sections that follow: an `AdminTo` edge is a lateral-execution opportunity (pass-the-hash to run code), a `GenericAll`/`WriteDacl` edge is DACL abuse, a `HasSession` edge is a credential-dumping opportunity on that host, and an `AllowedToDelegate` edge is a Kerberos delegation path.

## Exploitation notes

- Collect from the least-privileged position that works; a single low-priv domain user can usually collect objects, ACLs, and trusts (the high-value structure) even without session data.
- Session data is time-sensitive, so recollect it before planning lateral movement rather than trusting a stale snapshot.
- The graph tells you the path; the per-edge pages tell you how to walk each step.

## Tools

- **BloodHound CE**: the graph interface and path queries.
- **SharpHound**: the Windows/.NET collector.
- **bloodhound-python**: the network-based collector for non-Windows operators.

## References

- SpecterOps: BloodHound documentation
- The Hacker Recipes: BloodHound
