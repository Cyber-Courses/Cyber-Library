---
title: "Tactical Threat Intelligence: IOCs, TTPs, and SOC Support"
description: "How tactical intelligence delivers immediate, actionable detail to frontline defenders. A reference overview of the most granular level of cyber threat intelligence."
keywords:
  - tactical threat intelligence
  - indicators of compromise
  - tactics techniques procedures
  - soc threat intelligence
  - detection engineering
  - frontline defense intelligence
---

# Tactical Intelligence

Tactical intelligence is the most granular and immediate level of cyber threat intelligence. It delivers the concrete detail that frontline defenders use to detect and block activity: indicators of compromise such as file hashes, domains, and addresses, along with the tactics, techniques, and procedures that describe how adversaries behave. Its audience is the security operations center, detection engineers, and analysts who act in minutes and hours rather than weeks.

Within the three levels of intelligence, tactical work is the layer closest to the keyboard. It feeds detection rules, alerting logic, and triage decisions, and it often arrives in machine-readable form so tools can consume it directly. Frameworks such as MITRE ATT&CK help analysts organize techniques consistently, which makes tactical intelligence easier to map to defensive coverage and to share across teams.

Tactical intelligence matters because it shortens the time between an adversary's action and a defender's response. It supports faster detection, more accurate triage, and better prioritization of alerts. Its sources include telemetry, malware analysis output, sandbox results, and shared indicator feeds. Its central limitation is short shelf life: indicators change quickly, and low-context feeds can generate noise, so tactical intelligence works best when tied to the broader behavior described at the operational level.

## References

- MITRE ATT&CK framework documentation
- FIRST Traffic Light Protocol and information sharing guidance
