---
title: "S3"
order: 1
description: "Attaquer S3 : découvrir les buckets et objets, exploiter l'accès public et les ACL, et abuser des bucket policies accessibles en écriture ou en lecture."
keywords:
  - S3
  - bucket
  - public access
  - bucket policy
  - ACL
---

# S3

S3 stocke des objets dans des **buckets** nommés globalement, et l'espace de noms global est la première faiblesse : un nom de bucket est devinable, se résout via DNS et vous indique son existence avant même que vous ne déteniez une identité. L'accès est décidé par trois contrôles qui se chevauchent (le paramètre Block Public Access du compte, la bucket policy, et les ACL d'objets et de buckets), et une mauvaise configuration de l'un d'eux expose des données ou accorde l'écriture.

## Ce qui est regroupé ici

- **[Énumération](enumeration.md)** : trouver les buckets et objets par nommage, DNS et sondage des permissions.
- **[Accès public](public-access.md)** : buckets laissés en lecture ou en écriture ouverts à tous par des ACL ou par un Block Public Access désactivé.
- **[Bucket policy et ACL](bucket-policy-and-acl.md)** : policies et ACL trop larges accordant la lecture, l'écriture ou la prise de contrôle entre comptes.

Un bucket accessible en écriture qui sous-tend un site web ou une distribution logicielle se transforme en exécution de code en aval, et un nom de bucket libéré par une ressource supprimée est récupérable, ce qui est traité sous [prise de contrôle de DNS pendant](../../networking/dangling-dns-takeover.md).

## Références

- [HackTricks Cloud : énumération S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [PayloadsAllTheThings : AWS S3](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cloud%20-%20AWS%20Pentest.md)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [GrayhatWarfare : buckets publics](https://buckets.grayhatwarfare.com/)
- [S3Scanner](https://github.com/sa7mon/S3Scanner)
- [cloud_enum](https://github.com/initstring/cloud_enum)
