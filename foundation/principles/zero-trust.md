---
title: "Zero Trust: Verify Every Request, Trust No Location"
description: "The security model that treats no network location or identity as implicitly trustworthy and verifies every request before granting access."
keywords:
  - zero trust
  - zero trust architecture
  - never trust always verify
  - identity verification
  - microsegmentation
  - continuous authentication
---

# Zero trust

**Zero trust** is a security model built on a single rule: no user, device, or network location is trusted by default, and every request to access a resource must be verified before it is allowed. Its slogan is "never trust, always verify." Instead of asking "is this connection coming from inside our network?", a zero trust system asks "who is this, what device are they on, what are they asking for, and does that make sense right now?" and it asks that question again for every request.

## The castle and moat that stopped holding

Traditional enterprise security drew a hard line around the corporate network. A firewall guarded the edge, and anything inside the perimeter was treated as safe while anything outside was treated as hostile. Once a user or a machine got past the moat, it could usually reach most internal systems.

That model assumed people, applications, and data all lived in the same building. Remote work, cloud services, SaaS applications, and mobile devices broke that assumption: resources now sit across offices, homes, and multiple clouds, and a firewall at the edge cannot protect what lies outside it. The model also failed on its own terms, because a single stolen credential or compromised laptop on the inside inherited the trust of the whole network and could roam freely.

The idea of removing implicit trust took shape in response. Forrester analyst John Kindervag popularized the term "zero trust" around 2010, and Google's BeyondCorp project, described publicly in 2014, showed it working at scale: Google moved access controls off the network perimeter and onto individual users and devices, so employees could work from untrusted networks without a traditional VPN.

## Anatomy of a single access decision

The clearest way to see zero trust is to follow one request. Suppose an engineer opens the internal payroll dashboard from a laptop at a coffee shop.

1. **The request reaches an enforcement point, not the application.** A policy enforcement point (a gateway, proxy, or agent) sits in front of the payroll app. Nothing reaches the app directly, and being on a particular network grants nothing.
2. **Identity is proven.** The engineer authenticates, typically with multi-factor authentication. The service that will answer, and any service it calls, authenticates too.
3. **The device is examined.** Is this a managed laptop? Is its operating system patched, its disk encrypted, its endpoint protection running? A valid user on a compromised device is still a risk.
4. **Context is weighed.** Location, time of day, and behavior are compared with what is normal for this user. Logging in from two continents within an hour, for example, raises the risk score.
5. **The policy engine decides, for this request only.** It checks whether this identity, on this device, in this context, should reach this specific resource. Engineers may not need payroll data at all, so following [least privilege](least-privilege.md) the answer may simply be no.
6. **Trust keeps being reassessed.** If access is granted, it is scoped to what the request needs, and the session is re-evaluated as conditions change. If the device falls out of compliance or behavior turns anomalous mid-session, access can be cut.

NIST SP 800-207 names the core logical components in this flow: the **policy engine** makes the decision, the **policy administrator** carries it out by opening or closing the path, and the **policy enforcement point** sits in the data path and actually allows or blocks the traffic. The underlying mechanics of steps 2 and 5 are covered in [authentication and authorization](authentication-and-authorization.md).

## The signals behind "verify"

"Verify" means more than checking a password. A zero trust policy combines several kinds of evidence, each of which catches something the others miss.

| Signal | What is checked | What it catches |
|---|---|---|
| Identity | Who the user or service is, proven with strong, usually multi-factor, authentication | Guessed or phished passwords used alone |
| Device posture | Patch level, encryption, management status, endpoint protection | A legitimate user on an infected or unmanaged machine |
| Context and behavior | Location, time, request patterns compared with a baseline | Stolen sessions and credentials used in unusual ways |
| Resource sensitivity | How critical the requested data or system is | Low-assurance access to high-value targets |

Two structural techniques make these checks meaningful. **Microsegmentation** divides the network into small zones so that reaching one system does not grant reach to the rest, which is what keeps an attacker who gets in from finding many places to go. **Continuous evaluation** means trust is never a one-time stamp at login: it is reassessed throughout the session. Together they turn zero trust into a breach containment strategy, adding identity-based layers on top of the network and host controls described in [defense in depth](defense-in-depth.md).

## From NIST guidance to a federal deadline

Zero trust moved from vendor vocabulary to formal guidance through a short sequence of U.S. government documents:

- **August 2020: NIST SP 800-207.** NIST published *Zero Trust Architecture*, which defines the concept, sets out its tenets, and describes deployment models. It remains the most cited reference definition.
- **January 2022: OMB M-22-09.** The Office of Management and Budget directed federal civilian agencies to adopt a zero trust strategy, with specific goals for identity, devices, networks, applications, and data to be met by the end of fiscal year 2024.
- **April 2023: CISA Zero Trust Maturity Model 2.0.** CISA organized the work into five pillars (identity, devices, networks, applications and workloads, and data) plus three cross-cutting capabilities (visibility and analytics, automation and orchestration, and governance). Each pillar is graded across four stages: traditional, initial, advanced, and optimal.

The maturity model matters beyond government because it frames zero trust as a journey measured pillar by pillar, not a switch an organization flips. Many private companies and other governments use it as a roadmap.

## Where zero trust programs stall

- **Buying a box labeled "zero trust."** Zero trust is an architecture and a mindset, not a single product that can be purchased and switched on. Tools help, but only inside a design that removes implicit trust everywhere.
- **Stopping at the login.** Verifying identity once and then trusting the whole session quietly reintroduces the implicit trust the model is meant to remove.
- **Checking the user but not the device.** Strong authentication from a compromised or unpatched machine still hands an attacker a way in.
- **Adding friction without thought.** Controls that interrupt users constantly push them toward workarounds, undermining the security they were meant to add. Good designs keep checks invisible when risk is low and step up only when it rises.
- **Retrofitting instead of designing in.** Verify-first assumptions are far easier to build into new systems than to bolt onto legacy applications that expect a trusted internal network, which is why zero trust pairs naturally with [security by design](security-by-design.md).

## References

- [NIST SP 800-207, Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final)
- [NIST Glossary: Zero Trust](https://csrc.nist.gov/glossary/term/zero_trust)
- [CISA: Zero Trust Maturity Model](https://www.cisa.gov/zero-trust-maturity-model)
- [OMB M-22-09, Moving the U.S. Government Toward Zero Trust Cybersecurity Principles](https://www.whitehouse.gov/wp-content/uploads/2022/01/M-22-09.pdf)
- [Google Research: BeyondCorp, A New Approach to Enterprise Security (2014)](https://research.google/pubs/beyondcorp-a-new-approach-to-enterprise-security/)
