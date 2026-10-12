---
title: "Escalade de privilèges"
order: 2
description: "Le catalogue complet des chemins d'escalade de privilèges IAM d'AWS : iam:PassRole vers un service, manipulation de politiques et d'identifiants, réécriture de politiques de confiance et de ressources existantes, et abus de l'appartenance à des groupes."
keywords:
  - escalade de privilèges IAM
  - PassRole
  - iam-vulnerable
  - manipulation de politiques
  - politique de confiance
---

# Escalade de privilèges

L'escalade de privilèges AWS est un problème d'autorisations. Un principal qui détient l'une d'un ensemble connu d'actions dangereuses peut se promouvoir au rang d'administrateur sans toucher à un hôte. Le catalogue ci-dessous est l'union des chemins **iam-vulnerable** de BishopFox et de la liste de méthodes de Rhino Security Labs, regroupés par la primitive que chacun exploite.

## Les chemins

- **[PassRole](pass-role/index.md)** : `iam:PassRole` combiné à une action de création de service (EC2, Lambda, Glue, CloudFormation, Data Pipeline, CodeBuild, SageMaker) pour exécuter du code en tant que rôle privilégié.
- **[Manipulation de politiques](policy-manipulation/index.md)** : éditer directement une politique IAM, via de nouvelles versions de politique par défaut et l'attachement ou l'inlining de politiques d'utilisateur, de groupe et de rôle.
- **[Création d'identifiants](credential-creation/index.md)** : générer de nouveaux identifiants pour un principal sur lequel vous pouvez écrire, via des clés d'accès et des profils de connexion console.
- **[Ressources existantes](existing-resources/index.md)** : escalader via des ressources qui s'exécutent déjà avec un rôle privilégié, en mettant à jour des stacks, des dev endpoints et du code de fonction, ou en générant des URL présignées et des accès Instance Connect.
- **[Appartenance à un groupe](group-membership.md)** : `iam:AddUserToGroup` pour rejoindre un groupe qui porte plus d'autorisations.
- **[Politique de confiance](trust-policy.md)** : réécrire l'`AssumeRolePolicyDocument` d'un rôle pour pouvoir l'endosser.

## Choisir un chemin

Les chemins les plus discrets ne requièrent aucune nouvelle ressource : un `CreatePolicyVersion` ou un `AttachUserPolicy` modifie le principal sur place. Les chemins qui créent une ressource (une instance EC2, une Lambda) sont plus bruyants mais fonctionnent lorsque le seul levier que vous détenez est `PassRole` plus une action d'exécution. PMapper et le `iam__privesc_scan` de Pacu énumèrent lesquels de ces chemins le principal courant peut atteindre.

## Références

- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Rhino Security Labs: 21 ways to escalate privileges in AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/index.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
