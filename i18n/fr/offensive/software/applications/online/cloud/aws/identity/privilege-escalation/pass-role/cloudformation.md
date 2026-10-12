---
title: "CloudFormation : PassRole vers un rôle de service de stack"
description: "iam:PassRole vers une stack CloudFormation pour créer des ressources et exécuter des actions sous un rôle de stack privilégié."
keywords:
  - PassRole
  - CloudFormation
  - rôle de stack
  - CreateStack
  - déploiement
---

# CloudFormation

CloudFormation peut déployer une stack en utilisant un **rôle de service** plutôt que vos propres autorisations. Avec `iam:PassRole` et `cloudformation:CreateStack`, vous passez un rôle privilégié et soumettez un modèle qui provisionne tout ce que ce rôle peut créer, y compris de nouveaux principaux ou politiques IAM.

## Déployer avec un rôle passé

```bash
# template.yaml creates, for example, an admin user or attaches a policy
aws cloudformation create-stack --stack-name x \
  --template-body file://template.yaml \
  --role-arn arn:aws:iam::<acct>:role/<privileged-stack-role> \
  --capabilities CAPABILITY_NAMED_IAM
```

Comme c'est le rôle de stack qui réalise la création de ressources, un modèle qui crée un `AWS::IAM::User` avec une clé d'accès, ou attache `AdministratorAccess`, réussit même si votre propre principal ne le peut pas.

## Notes d'exploitation

- `CAPABILITY_NAMED_IAM` est requis lorsque le modèle touche à IAM ; c'est le rôle, et non vous, qui doit détenir les autorisations IAM.
- Mettre à jour une stack existante qui porte déjà un rôle privilégié est la variante [ressources existantes](../existing-resources/update-stack.md) et ne nécessite aucun `PassRole`.
- Les sorties de la stack peuvent renvoyer directement la clé d'accès créée, de sorte qu'aucune étape de récupération séparée n'est nécessaire.

## Outils

- **AWS CLI** (`cloudformation create-stack`).
- **Pacu** : modules privesc et données de CloudFormation.

## Références

- [Rhino Security Labs: AWS privilege escalation (CloudFormation)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
