---
title: "CodeStar : PassRole vers CreateProject et auto-ajout de membre d'équipe"
description: "iam:PassRole avec codestar:CreateProject ou CreateProjectFromTemplate pour s'exécuter sous le rôle de service CodeStar, et codestar:AssociateTeamMember pour s'ajouter soi-même comme propriétaire de projet."
keywords:
  - CodeStar
  - CreateProject
  - AssociateTeamMember
  - PassRole
  - rôle de service
  - escalade de privilèges
---

# CodeStar

CodeStar porte deux chemins d'escalade distincts. Le premier est un chemin `PassRole` : `codestar:CreateProject` (ou `CreateProjectFromTemplate`) avec `iam:PassRole` provisionne un projet dont le modèle CloudFormation s'exécute en tant que rôle de service passé, exécutant des ressources définies par l'attaquant sous ce rôle. Le second ne nécessite aucun `PassRole` : `codestar:AssociateTeamMember` vous permet de vous ajouter comme **Owner** d'un projet CodeStar, héritant de l'accès du rôle du projet.

## CreateProject avec un rôle passé

```bash
aws codestar create-project --name x --id x \
  --source-code ... --toolchain '{"roleArn":"arn:aws:iam::<acct>:role/<privileged-role>","stackParameters":{}}'
```

La stack de la chaîne d'outils est déployée avec le rôle que vous passez, de sorte qu'un modèle qui crée une politique admin ou une clé d'accès s'exécute avec les droits de ce rôle.

## S'auto-ajouter comme propriétaire de projet

```bash
# grant yourself Owner on an existing project, no PassRole needed
aws codestar associate-team-member --project-id <proj> \
  --user-arn arn:aws:iam::<acct>:user/<me> --project-role Owner --remote-access-allowed
```

Les politiques gérées propres à CodeStar étendent ensuite à vous l'accès aux ressources du projet.

## Notes d'exploitation

- `CreateProjectFromTemplate` (l'API la plus ancienne) est la variante que la plupart des politiques IAM autorisent encore, et celle qu'utilisait la recherche d'origine.
- L'auto-ajout en tant qu'Owner est le plus discret des deux : il modifie une appartenance, pas une infrastructure, et réutilise le rôle existant du projet au lieu d'en passer un nouveau.
- CodeStar est déprécié par AWS mais encore présent dans de nombreux comptes de longue durée, précisément là où ces attributions trop larges persistent.

## Outils

- **AWS CLI** (`codestar create-project` / `create-project-from-template` / `associate-team-member`).
- **Pacu** (`iam__privesc_scan`) : énumère la surface capable de PassRole.

## Références

- [Rhino Security Labs: AWS privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
