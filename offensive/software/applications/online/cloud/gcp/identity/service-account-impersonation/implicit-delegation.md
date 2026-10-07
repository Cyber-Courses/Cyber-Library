---
title: "Implicit delegation: chain impersonation through an intermediary"
order: 2
description: "Chaining impersonation through an intermediate service account with iam.serviceAccounts.implicitDelegation to reach a target you cannot call directly."
keywords:
  - implicit delegation
  - delegation chain
  - service account
  - impersonation
  - getAccessToken
---

# Implicit delegation

When you cannot impersonate a target service account directly but you can impersonate an intermediate account that holds Token Creator on the target, **delegation** chains through it. You ask IAM Credentials to mint the target's token while delegating through the intermediary, reaching an account one hop away.

## Chain through a delegate

```bash
# you can impersonate SA-A; SA-A has Token Creator on SA-B (the real target)
gcloud auth print-access-token \
  --impersonate-service-account <sa-b>@<proj>.iam.gserviceaccount.com \
  --delegates <sa-a>@<proj>.iam.gserviceaccount.com
```

The `generateAccessToken` API takes a `delegates` list, so longer chains (A to B to C) work as long as each link holds Token Creator on the next.

## Exploitation notes

- Map the Token Creator graph from the IAM policies: delegation turns a chain of weak grants into a path to a strong account.
- Delegation is easy to miss in review because no single binding names you on the final target.
- Each hop is a short-lived token, so re-mint as the chain is traversed.

## Tools

- **gcloud** (`--impersonate-service-account` with `--delegates`).
- IAM-graph analysis to find delegation chains.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: delegated credentials](https://cloud.google.com/iam/docs/create-short-lived-credentials-delegated)
