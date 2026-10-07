---
title: "Detection Deception: Honeypots, Decoys, and Canaries"
order: 4
description: "How deception uses honeypots, decoys, and canary tokens to produce high-confidence signals of intrusion."
keywords:
  - cyber deception
  - honeypots
  - decoy systems
  - canary tokens
  - high-confidence alerts
  - intrusion detection
---

# Deception

Deception is the practice of placing honeypots, decoys, and canaries throughout an environment so that any interaction with them signals likely malicious activity. Because legitimate users and systems have no reason to touch these planted assets, a single interaction becomes a high-confidence indicator worth immediate attention.

Within the Detection phase, deception offers a different kind of signal from most analytics. Conventional detection sifts real activity to find the suspicious fraction, which inevitably produces false positives. Deception inverts the problem: the decoy exists only to be triggered by someone exploring where they should not, so the resulting alert carries very low noise.

In practice, deception ranges from fake credentials and documents to entire decoy hosts and services that mimic production systems. Canary tokens embedded in files, directories, or API keys fire when accessed, revealing reconnaissance or data theft. Effective deception is believable and well placed, blending into the environment so adversaries cannot easily distinguish it from real targets. Because the signals are so clean, deception pairs well with automated response and with hunting that investigates how an intruder reached the decoy.

## References

- MITRE Engage, Adversary Engagement framework
- NIST SP 800-160 Volume 2, Developing Cyber-Resilient Systems
