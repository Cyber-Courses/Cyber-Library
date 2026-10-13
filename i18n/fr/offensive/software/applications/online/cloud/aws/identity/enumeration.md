---
title: "Énumération : cartographier le graphe IAM et vos permissions effectives"
order: 1
description: "Cartographier les principaux, stratégies et relations de confiance IAM d'un compte AWS depuis un point d'ancrage : appels iam:List et Get, enumerate-iam, Pacu et PMapper pour trouver les permissions effectives et les rôles atteignables."
keywords:
  - énumération IAM
  - Pacu
  - PMapper
  - enumerate-iam
  - permissions effectives
---

# Énumération

Chaque élévation et pivot dans AWS part d'une seule question : **que peut réellement faire le principal que je détiens, et que peut-il atteindre.** Les réponses IAM ne sont pas locales, de sorte que l'énumération est un ensemble d'appels d'API contre le service IAM et une analyse de graphe sur le résultat.

## Qui suis-je

```bash
aws sts get-caller-identity            # account id, principal ARN, user vs assumed-role
aws iam get-user                       # if a user; fails for role sessions
```

## Lire le graphe IAM

Avec `iam:List*`/`iam:Get*`, récupérez l'ensemble du panorama des stratégies :

```bash
aws iam list-users ; aws iam list-roles ; aws iam list-groups
aws iam list-attached-user-policies --user-name <u>
aws iam get-account-authorization-details > iam.json   # the whole graph in one call
```

`get-account-authorization-details` est le prix : chaque principal, chaque stratégie inline et attachée, et chaque document de confiance en une seule réponse, ce qui est exactement ce que consomment les outils de graphe.

## Résoudre les permissions effectives sans droits de lecture

Les permissions ne sont souvent pas lisibles même lorsqu'elles sont utilisables. Deux approches :

```bash
# Brute the API surface with harmless calls and record what is allowed
enumerate-iam --access-key AKIA... --secret-key ...

# Pacu's permission enumeration and privesc scan
pacu > run iam__enum_permissions ; run iam__privesc_scan

# where your own policies are readable, confirm what they grant without calling the actions
aws iam simulate-principal-policy --policy-source-arn <your-arn> \
  --action-names iam:CreateAccessKey sts:AssumeRole s3:GetObject
```

## Tracer les chemins vers l'administration

```bash
# PMapper: build the graph, then query for escalation paths
pmapper graph create
pmapper query 'preset privesc *'
pmapper visualize
```

## Notes d'exploitation

- Privilégiez `get-account-authorization-details` d'abord ; il condense des dizaines d'appels et alimente directement PMapper.
- Un `iam:Get*` refusé ne signifie pas que l'action est refusée : utilisez `enumerate-iam` pour découvrir la surface utilisable par sondage.
- Les sessions de rôle n'ont pas d'utilisateur ; reportez vos questions d'identité sur l'ARN du rôle assumé et ses stratégies de session.

## Outils

- **Pacu** (`iam__enum_permissions`, `iam__privesc_scan`) : énumération basée sur la session et détection d'élévation de privilèges.
- **PMapper** : construction du graphe IAM et requêtes de chemins vers l'administrateur.
- **CloudFox** : inventaire rapide, orienté attaquant, des principaux, rôles, secrets et ressources atteignables.
- **Cloudsplaining** : analyse de stratégies IAM pour les constats d'élévation de privilèges et d'exposition de ressources.
- **enumerate-iam** : découverte de la surface d'API autorisée par sondage.
- **ScoutSuite** / **Prowler** : posture à l'échelle du compte et inventaire des principaux.
- **weirdAAL** / **SkyArk** : découverte d'API du non authentifié au faible privilège et chasse aux administrateurs fantômes.

## Références

- [PMapper (NCC Group)](https://github.com/nccgroup/PMapper)
- [enumerate-iam (Andres Riancho)](https://github.com/andresriancho/enumerate-iam)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
