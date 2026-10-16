---
title: "Security by Design: Building Protection in From the Start"
description: "The practice of planning security into a system's architecture from the first design decisions rather than adding it after the system already works."
keywords:
  - security by design
  - secure by design
  - threat modeling
  - shift left security
  - secure development lifecycle
  - secure by default
---

# Security by design

**Security by design** is the practice of treating protection as a requirement that shapes a system from its first design decisions, not a layer added once the system already runs. When security is weighed alongside features, performance, and cost from the start, the architecture that results is usually simpler to defend and far less brittle than one patched after release. The opposite habit, shipping first and securing later, tends to leave gaps that are expensive to close and easy for an attacker to find.

## Shifting left: when the decisions get made

Most of a system's security is determined long before anyone writes exploit code against it. Choices about trust boundaries, data flows, default settings, and who can talk to what are made during design, and they are hard to reverse once code, data, and users depend on them. "Shifting left" means pulling security activity toward the earlier, cheaper end of the development timeline:

- **Requirements.** Security goals are written down as acceptance criteria, not wishes.
- **Design.** Architecture is reviewed for weak trust boundaries before implementation.
- **Build and test.** Automated checks catch regressions while a fix is still a small change.

The point is not to do security work once, early, and stop. It is to make the earliest decisions the load-bearing ones.

## Threat modeling before the first line of code

The core design-time activity is threat modeling: a structured look at what you are building, what can go wrong, and what you will do about it. A simple version answers four questions, often attributed to the team behind Microsoft's Security Development Lifecycle:

1. **What are we building?** A diagram of components, data stores, and the trust boundaries between them.
2. **What can go wrong?** Spoofing, tampering, information disclosure, and similar failure categories, walked through boundary by boundary.
3. **What are we going to do about it?** A mitigation for each threat worth addressing.
4. **Did we do a good job?** A review that checks the model against the system actually built.

Because it works on diagrams, threat modeling can expose a flawed trust assumption weeks before that assumption becomes code.

## The rising cost of a late fix

Studies of software defects have repeatedly found the same shape: the later a flaw is caught, the more it costs to remove, because more work has been built on top of it.

| Stage flaw is caught | What a fix touches | Relative cost |
|---|---|---|
| Design | A diagram or a decision | Lowest |
| Development | Code not yet shipped | Low |
| Testing | Code plus test suites | Moderate |
| Production | Live systems, data, and users | Highest |

A missing authorization check is a one-line note in a design review and a coordinated incident response once it is live. Security by design tries to catch it while it is still the one-line note.

## Secure by design is not the same as secure by default

The two ideas are often paired but are distinct. Secure by design is about the process, building protection in deliberately. Secure by default is about the shipped result: the product's out-of-the-box configuration is its safe one, so a user is protected without having to find and flip security switches. A system can be thoughtfully designed yet ship wide open, and the modern expectation, pushed by bodies like CISA, is that vendors deliver both.

## Where it leans on the other principles

Security by design is a frame that other principles fill in. A sound design assumes no single control is perfect and layers them, which is [defense in depth](defense-in-depth.md). It grants each component only what it needs, applying [least privilege](least-privilege.md), and it collects only the data it truly requires through [minimization](minimization.md). Deciding trust at design time rather than inheriting it from the network is the heart of [zero trust](zero-trust.md).

## References

- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [CISA, Secure by Design](https://www.cisa.gov/securebydesign)
- [Microsoft Security Development Lifecycle (SDL) Practices](https://www.microsoft.com/en-us/securityengineering/sdl/practices)
- [OWASP Developer Guide: Secure Design Principles](https://owasp.org/www-project-developer-guide/)
- [NIST SP 800-160 Vol. 1, Engineering Trustworthy Secure Systems](https://csrc.nist.gov/pubs/sp/800/160/v1/r1/final)
