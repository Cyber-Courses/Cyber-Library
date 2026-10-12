---
title: "Journalisation et détection"
order: 8
description: "Aveugler la détection Azure : altérer l'Activity Log, les diagnostic settings d'Azure Monitor, Defender for Cloud et Sentinel pour supprimer la journalisation et l'alerting."
keywords:
  - journalisation Azure
  - Activity Log
  - Azure Monitor
  - Defender for Cloud
  - Sentinel
  - évasion de détection
---

# Journalisation et détection

Azure enregistre l'activité du plan de contrôle dans l'**Activity Log** et déploie la détection via **Azure Monitor** (diagnostic settings vers Log Analytics), **Defender for Cloud** et **Microsoft Sentinel**. Un opérateur cherchant à rester invisible dégrade ces éléments par ordre de levier : arrêter l'export qui alimente le SIEM, rétrograder le service de posture qui lève les alertes, et désactiver les règles d'analytics qui se déclencheraient. L'Activity Log lui-même est conservé de façon immuable pendant 90 jours, donc le vrai jeu consiste à contrôler ce qui atteint les sinks de rétention longue durée et d'alerting, et à privilégier les opérations qui n'y atterrissent jamais.

## Pages

- **[Activity Log](activity-log.md)** : ce que le registre d'opérations de l'abonnement capture et ne capture pas, et couper son export vers les sinks en aval.
- **[Azure Monitor](azure-monitor.md)** : supprimer les diagnostic settings pour arrêter les logs des ressources, et voler les clés du workspace Log Analytics.
- **[Defender for Cloud](defender-for-cloud.md)** : rétrograder les plans et la policy pour aveugler la détection de menaces cloud.
- **[Sentinel](sentinel.md)** : désactiver les règles d'analytics et retirer les connecteurs de données pour aveugler le SIEM.

## Références

- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Stratus Red Team : techniques Azure](https://stratus-red-team.cloud/attack-techniques/azure/)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
