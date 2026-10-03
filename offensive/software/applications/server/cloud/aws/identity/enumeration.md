---
title: "Enumeration: mapping the IAM graph and your effective permissions"
description: "Mapping an AWS account's IAM principals, policies, and trust relationships from a foothold: iam:List and Get calls, enumerate-iam, Pacu, and PMapper to find effective permissions and reachable roles."
keywords:
  - IAM enumeration
  - Pacu
  - PMapper
  - enumerate-iam
  - effective permissions
---

# Enumeration

Every escalation and pivot in AWS starts from one question: **what can the principal I hold actually do, and what can it reach.** IAM answers are not local, so enumeration is a set of API calls against the IAM service and a graph analysis on the result.

## Who am I

```bash
aws sts get-caller-identity            # account id, principal ARN, user vs assumed-role
aws iam get-user                       # if a user; fails for role sessions
```

## Reading the IAM graph

With `iam:List*`/`iam:Get*`, pull the whole policy picture:

```bash
aws iam list-users ; aws iam list-roles ; aws iam list-groups
aws iam list-attached-user-policies --user-name <u>
aws iam get-account-authorization-details > iam.json   # the whole graph in one call
```

`get-account-authorization-details` is the prize: every principal, inline and attached policy, and trust document in a single response, which is exactly what graph tools consume.

## Resolving effective permissions without read rights

Permissions are often not readable even when they are usable. Two approaches:

```bash
# Brute the API surface with harmless calls and record what is allowed
enumerate-iam --access-key AKIA... --secret-key ...

# Pacu's permission enumeration and privesc scan
pacu > run iam__enum_permissions ; run iam__privesc_scan
```

## Graphing paths to admin

```bash
# PMapper: build the graph, then query for escalation paths
pmapper graph create
pmapper query 'preset privesc *'
pmapper visualize
```

## Exploitation notes

- Prefer `get-account-authorization-details` first; it collapses dozens of calls and feeds PMapper directly.
- A denied `iam:Get*` does not mean the action is denied: use `enumerate-iam` to learn the usable surface by probing.
- Role sessions have no user; pivot your identity questions to the assumed-role ARN and its session policies.

## Tools

- **Pacu** (`iam__enum_permissions`, `iam__privesc_scan`): session-based enumeration and privesc detection.
- **PMapper**: IAM graph construction and path queries to administrator.
- **enumerate-iam**: probe-based discovery of the allowed API surface.
- **ScoutSuite** / **Prowler**: account-wide posture and principal inventory.

## References

- [PMapper (NCC Group)](https://github.com/nccgroup/PMapper)
- [enumerate-iam (Andres Riancho)](https://github.com/andresriancho/enumerate-iam)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
