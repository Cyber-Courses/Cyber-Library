---
title: "Tactical Threat Intelligence: TTPs and Defensive Detection"
description: "How tactical intelligence describes adversary tactics, techniques, and procedures so frontline defenders can detect and respond. A reference overview within the four levels of cyber threat intelligence."
keywords:
  - tactical threat intelligence
  - tactics techniques procedures
  - adversary behavior
  - soc threat intelligence
  - detection engineering
  - frontline defense intelligence
---

# Tactical Intelligence

Tactical intelligence describes how adversaries operate: the tactics, techniques, and procedures they use to gain access, move through an environment, and reach their objectives. It gives frontline defenders the behavioral detail needed to build detections and shape response. It sits just above technical intelligence, which supplies the most atomic and short lived indicators such as hashes, addresses, and domains. Its audience is the security operations center, detection engineers, and analysts who act in minutes and hours rather than weeks.

Within the four levels of intelligence, tactical work is among the layers closest to the keyboard. It feeds detection rules, hunting hypotheses, and triage decisions, and it is most useful when expressed as repeatable adversary behavior rather than one-off artifacts. Frameworks such as MITRE ATT&CK help analysts organize techniques consistently, which makes tactical intelligence easier to map to defensive coverage and to share across teams.

Tactical intelligence matters because it shortens the time between an adversary's action and a defender's response, and because behavior is harder for an adversary to change than an individual indicator. It supports more durable detection, more accurate triage, and better prioritization of alerts. Its sources include telemetry, malware and tooling analysis, incident reporting, and structured technique libraries. Its central limitation is that techniques still evolve and can be obscured, so tactical intelligence works best when tied to the broader campaign context described at the operational level and enriched with the concrete indicators from the technical level.

## References

- MITRE ATT&CK framework documentation
- FIRST Traffic Light Protocol and information sharing guidance
