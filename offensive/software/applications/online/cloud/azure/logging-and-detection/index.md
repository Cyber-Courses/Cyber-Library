---
title: "Logging and detection"
description: "Blinding Azure detection: tampering with the Activity Log, Azure Monitor diagnostic settings, Defender for Cloud, and Sentinel to suppress logging and alerting."
keywords:
  - Azure logging
  - Activity Log
  - Azure Monitor
  - Defender for Cloud
  - Sentinel
  - detection evasion
---

# Logging and detection

Azure records control-plane activity in the **Activity Log** and fans detection out through **Azure Monitor** (diagnostic settings into Log Analytics), **Defender for Cloud**, and **Microsoft Sentinel**. An operator working to stay unseen degrades these in order of leverage: stop the export that feeds the SIEM, downgrade the posture service that raises alerts, and disable the analytics rules that would fire. The Activity Log itself is retained immutably for 90 days, so the real game is controlling what reaches long-term and alerting sinks, and preferring operations that never land there.

## Pages

- **[Activity Log](activity-log.md)**: what the subscription operation log does and does not capture, and cutting its export to downstream sinks.
- **[Azure Monitor](azure-monitor.md)**: deleting diagnostic settings to stop resource logs, and stealing Log Analytics workspace keys.
- **[Defender for Cloud](defender-for-cloud.md)**: downgrading plans and policy to blind cloud threat detection.
- **[Sentinel](sentinel.md)**: disabling analytics rules and removing data connectors to blind the SIEM.

## References

- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Stratus Red Team: Azure techniques](https://stratus-red-team.cloud/attack-techniques/azure/)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
