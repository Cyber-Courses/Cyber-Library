---
title: "URL présignée : distribuer un accès à durée limitée sous vos autorisations"
order: 1
description: "Générer des URL présignées pour des objets S3 et d'autres ressources afin de distribuer un accès à durée limitée sous vos propres autorisations."
keywords:
  - URL présignée
  - S3
  - durée limitée
  - requête signée
  - accès
---

# URL présignée

Une URL présignée intègre une signature faite avec vos identifiants, accordant à quiconque détient l'URL l'action signée pendant toute sa durée de vie. Vous générez un accès présigné à des objets que votre principal peut atteindre, puis le partagez ou le placez où il faut, exfiltrant des données sans que le destinataire ne détienne aucun identifiant AWS.

## Présigner un objet S3

```bash
aws s3 presign s3://<bucket>/<key> --expires-in 604800
# the returned URL downloads the object with no credentials, for 7 days
```

## Notes d'exploitation

- L'URL porte vos autorisations, pas celles du destinataire, ce qui en fait une primitive nette d'exfiltration et de partage.
- Les URL présignées de SageMaker et d'autres services fonctionnent de la même façon : un lien signé et sans identifiant vers une action privilégiée.
- La signature reste valide jusqu'à son expiration ou la révocation de vos identifiants : traitez donc les URL émises comme des secrets actifs.

## Outils

- **AWS CLI** (`s3 presign`), **boto3** (`generate_presigned_url`) pour des opérations arbitraires.

## Références

- [AWS: using presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)
- [HackTricks Cloud: AWS S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
