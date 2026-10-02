---
title: "Preparation Hardening: Reducing Attack Surface with Baselines"
description: "How hardening reduces attack surface by applying secure baselines and benchmarks to systems and services."
keywords:
  - system hardening
  - attack surface reduction
  - secure baselines
  - security benchmarks
  - secure configuration
  - least functionality
---

# Hardening

Hardening is the practice of reducing a system's attack surface by removing unnecessary features and applying secure configuration baselines. The goal is to leave only what is needed for a system to do its job, so there are fewer services, accounts, and settings an adversary can abuse.

Within the Preparation phase, hardening is proactive defense built into the systems themselves. Rather than relying solely on detection and response after the fact, hardening removes opportunities before they can be used. A well-hardened estate raises the effort an attacker must spend and shrinks the paths available to them, which benefits every later phase of defense.

In practice, hardening applies recognized baselines and benchmarks to operating systems, applications, cloud services, and network devices. Teams disable unused services and ports, enforce least functionality and least privilege, remove default credentials, and tighten settings for authentication, logging, and encryption. Benchmarks such as those published by the Center for Internet Security provide concrete, testable configurations, and automated tooling checks systems against them continuously. Because environments drift, hardening is maintained over time rather than applied once, keeping systems close to their secure baseline as they change.

## References

- CIS Benchmarks, secure configuration guidance
- NIST SP 800-123, Guide to General Server Security
