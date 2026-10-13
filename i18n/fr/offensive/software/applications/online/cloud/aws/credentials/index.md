---
order: 2
title: "Credentials AWS"
description: "Récolter des credentials AWS : clés d'accès à long terme, jetons de session STS, le service de métadonnées d'instance EC2 et ECS (IMDSv1/v2 et SSRF), Secrets Manager, Parameter Store et KMS, ainsi que les courtiers de credentials de service qui les émettent."
keywords:
  - credentials AWS
  - clés d'accès
  - STS
  - IMDS
  - Secrets Manager
  - Parameter Store
  - KMS
---

# Credentials

Chaque appel d'API AWS nécessite un credential, l'obtenir est donc le premier geste et en récolter davantage est la façon dont une mission s'élargit. Les credentials se présentent sous forme de **clés d'accès** à longue durée de vie, de **jetons de session STS** temporaires (une clé d'accès, un secret, un jeton de session et une expiration), et de credentials de rôle livrés par le **service de métadonnées d'instance**. Chacun a des caractéristiques de vol et de réutilisation différentes, et plusieurs services AWS renvoient de nouveaux credentials lorsqu'ils sont lus.

## Ce qui s'y rattache

- **[Clés d'accès](access-keys.md)** : paires `AKIA` à long terme dans les fichiers, les variables d'environnement, la config CI et l'historique du code source.
- **[Métadonnées d'instance](instance-metadata/index.md)** : credentials de rôle depuis IMDS sur EC2 et ECS, y compris le flux de jeton IMDSv2 et la récupération par SSRF.
- **[Jetons STS](sts-tokens.md)** : capturer et rejouer des jetons de session à courte durée de vie.
- **[Magasins de secrets](secret-stores/index.md)** : Secrets Manager, SSM Parameter Store et KMS comme sources de credentials.
- **[Courtiers de credentials](credential-brokers.md)** : services qui émettent des credentials pour d'autres principaux.

Les pages IMDS sont le versant AWS de la technique web de [server-side request forgery](../../../../server/web/code/injection/request-forgery/index.md), référencée en renvoi plutôt que dupliquée.

## Références

- [HackTricks Cloud: AWS metadata and credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: EC2 instance metadata service (IMDSv2)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
