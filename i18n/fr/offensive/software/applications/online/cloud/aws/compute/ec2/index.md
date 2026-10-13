---
order: 1
title: "EC2"
description: "Attaquer les instances EC2 : secrets et code dans les user data, le profil d'instance attaché et son rôle, et la récupération de données depuis les snapshots EBS et les AMI partagées."
keywords:
  - EC2
  - user data
  - profil d'instance
  - snapshots
  - AMI
---

# EC2

EC2 est la surface de calcul la plus riche d'AWS parce qu'une instance relie entre eux un **rôle** (le profil d'instance), un **script de démarrage** (user data), et des **disques persistants** (volumes EBS, ainsi que les AMI et snapshots dont ils proviennent). Chacun est une voie d'entrée distincte : voler le rôle depuis le service de métadonnées, lire les secrets laissés dans les user data, ou récupérer les données d'une autre instance depuis un snapshot que vous pouvez monter.

## Ce qui est regroupé ici

- **[User data](user-data.md)** : lire les scripts de démarrage pour y trouver des secrets, ou définir les user data pour exécuter du code au prochain démarrage.
- **[Profil d'instance](instance-profile.md)** : récolter les identifiants du rôle attaché depuis le service de métadonnées.
- **[Snapshots](snapshots.md)** : créer, partager et monter des snapshots EBS pour lire des volumes hors ligne.
- **[AMI](ami.md)** : secrets intégrés dans des images machine partagées ou publiques, et lancement à partir d'elles.

Lancer une *nouvelle* instance avec un rôle passé est une voie d'élévation de privilèges et vit sous [identity PassRole](../../identity/privilege-escalation/pass-role/ec2.md) ; les pages ici couvrent les instances qui existent déjà.

## Références

- [HackTricks Cloud: AWS EC2 enumeration and privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Rhino Security Labs: AWS privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
