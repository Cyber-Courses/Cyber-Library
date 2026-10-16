---
title: "Zero Trust: Verify Every Request, Trust No Location"
description: "Zero trust is a security model that treats no network location or identity as implicitly trusted and verifies every request before granting access."
keywords:
  - zero trust
  - zero trust architecture
  - never trust always verify
  - policy enforcement point
  - microsegmentation
  - continuous verification
---

# Zero trust

**Zero trust** is a security model that grants no automatic trust based on where a request comes from. Being inside the corporate network, on a company laptop, or behind the firewall earns nothing on its own. Every attempt to reach a resource is treated as if it came from an open, hostile network and must prove its identity, its device posture, and its right to that specific resource before access is allowed. The phrase that captures it is "never trust, always verify."

## The perimeter model it replaced

For decades, enterprise security worked like a castle and moat. A firewall guarded the edge, everything outside was hostile, and everything inside was treated as safe. The flaw was structural: once an attacker got past the moat with one stolen password or one infected laptop, they inherited the network's trust and could move sideways to almost anything.

Remote work, cloud services, and personal devices then dissolved the moat entirely, because the people and data no longer sat in one building. The term "zero trust" was popularized by Forrester analyst John Kindervag around 2010, and Google's BeyondCorp project, published from 2014 onward, proved it at scale by moving access decisions off the network and onto individual users and devices.

## NIST's three logical components

NIST Special Publication 800-207 describes zero trust architecture as a split between a control plane that decides and a data plane that enforces. Three logical components carry the model:

| Component | Role |
|---|---|
| Policy Engine | Weighs identity, device state, and context, then decides to grant, deny, or revoke access |
| Policy Administrator | Executes the decision by opening or closing the communication path |
| Policy Enforcement Point | Sits in front of the resource and lets nothing through until the path is opened |

The subject is not one gateway but this decision loop, run for every request rather than once at login.

## Verifying a single request

Follow an engineer opening an internal payroll dashboard from a coffee shop.

1. The request hits a policy enforcement point, never the application directly. Sitting on any particular network grants nothing.
2. Identity is proven, usually with multi-factor authentication, drawing on [authentication and authorization](authentication-and-authorization.md).
3. The device is checked: is it managed, patched, and free of known compromise?
4. The policy engine scores the full context and grants access to that one resource, for that one session, at the minimum level needed, applying [least privilege](least-privilege.md).
5. The session is continuously re-evaluated. If the device falls out of compliance or the behavior turns anomalous, access is revoked mid-session.

## The price of removing implicit trust

Zero trust is not free. Every request now carries the cost of a policy lookup, which adds latency and makes the policy engine a critical dependency: if it is slow or offline, legitimate work stalls. Strong identity and device signals must exist and stay accurate, or decisions are made on bad data. Done poorly, the enforcement points become a new perimeter in disguise.

It also does not replace other defenses. Narrowing each session to one resource shrinks the reachable [attack surface](../concepts/assets-and-attack-surface.md), and zero trust pairs with [defense in depth](defense-in-depth.md) rather than substituting for it, while detailed per-request logs feed [accountability](accountability.md).

## References

- [NIST SP 800-207, Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final)
- [NIST SP 1800-35, Implementing a Zero Trust Architecture](https://www.nccoe.nist.gov/projects/implementing-zero-trust-architecture)
- [CISA Zero Trust Maturity Model](https://www.cisa.gov/zero-trust-maturity-model)
- [BeyondCorp: A New Approach to Enterprise Security](https://research.google/pubs/pub43231/)
