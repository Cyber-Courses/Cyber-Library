---
title: "Journalisation et détection AWS"
order: 8
description: "Aveugler la détection AWS : désactiver, détourner et contourner CloudTrail, GuardDuty, Config et CloudWatch pour supprimer la journalisation et l'alerte."
keywords:
  - CloudTrail
  - GuardDuty
  - AWS Config
  - CloudWatch
  - log tampering
  - detection evasion
  - anti-forensics
---

# Journalisation et détection

Le plan d'audit est une cible, pas une contrainte. Avant des actions bruyantes, ou pour en effacer la trace après coup, les services de journalisation et de détection du compte sont arrêtés, restreints, détournés ou supprimés. AWS centralise cela dans quatre services : **CloudTrail** enregistre les appels d'API, **Config** enregistre l'état des ressources, **GuardDuty** note le comportement à la recherche de menaces, et **CloudWatch** détient les logs, les métriques et les alarmes qui alertent un répondant. Dégrader l'un d'eux élargit ce qui peut être fait sans être vu.

Les manœuvres les plus discrètes sont sélectives (restreindre les event selectors d'un trail, archiver une catégorie de findings) plutôt que destructrices (supprimer le trail), parce que les appels d'arrêt et de suppression sont eux-mêmes enregistrés : ils ne sont donc propres que s'ils atterrissent avant que l'enregistrement n'atteigne un endroit que vous ne contrôlez pas.

## Ce qui est regroupé ici

- **[CloudTrail](cloudtrail.md)** : arrêter, supprimer et détourner les trails, et supprimer les événements de management et de données.
- **[GuardDuty](guardduty.md)** : désactiver les détecteurs et auto-archiver les findings, et rester dans les angles morts des analyses.
- **[Config](config.md)** : arrêter les configuration recorders et supprimer les règles et delivery channels.
- **[CloudWatch](cloudwatch.md)** : supprimer les log groups, metric filters et alarmes, et désactiver les actions d'alarme.
- **[Services de sécurité](security-services.md)** : désactiver Macie, Security Hub, Detective et Inspector, la couche de posture et d'agrégation des findings.

## Références

- [HackTricks Cloud : contournement de défense AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [Stratus Red Team : contournement de défense AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [AWS : concepts CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-concepts.html)
- [Datadog Security Labs : recherche sur le contournement de défense AWS](https://securitylabs.datadoghq.com/)
