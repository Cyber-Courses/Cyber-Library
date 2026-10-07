---
title: "Detection Improvement: Measuring Coverage and Tuning Over Time"
order: 7
description: "How detection improvement measures coverage and tunes detections over time, often using frameworks such as MITRE ATT&CK."
keywords:
  - detection improvement
  - detection coverage
  - mitre attack coverage
  - detection tuning
  - metrics and measurement
  - continuous improvement
---

# Improvement

Improvement is the ongoing effort to measure how well detection works and to make it better over time. It asks which adversary behaviors the program can actually see, where the gaps are, and whether existing detections still perform, then directs effort toward the weaknesses that matter most.

Within the Detection phase, improvement is the feedback loop that keeps the rest honest. Collection, analysis, alerting, and engineering all benefit from knowing where coverage is thin and where noise is high. Without measurement, a team cannot tell whether it is getting stronger or simply accumulating rules.

In practice, improvement often uses a common framework such as MITRE ATT&CK to map detections against techniques and visualize coverage. Teams track metrics like detection counts by tactic, false-positive rates, time to detect, and the outcomes of hunts and incidents. Findings from real incidents and purple-team exercises reveal missed activity, which becomes new or refined detections. The work is continuous: coverage is reviewed on a cadence, priorities shift with the threat landscape, and tuning balances sensitivity against noise so that improvement is measured, not assumed.

## References

- MITRE ATT&CK, coverage mapping guidance
- NIST SP 800-55, Performance Measurement for Information Security
