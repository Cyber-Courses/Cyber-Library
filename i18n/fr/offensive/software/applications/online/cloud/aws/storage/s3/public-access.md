---
title: "Accès public : lire et écrire dans des buckets exposés à tous"
order: 2
description: "Lecture et écriture d'objets dans des buckets laissés accessibles à tous par des ACL ou par la désactivation de Block Public Access."
keywords:
  - S3
  - public access
  - Block Public Access
  - ACL
  - anonymous
---

# Accès public

Un bucket est public lorsque **Block Public Access** est désactivé et que sa policy ou une ACL accorde l'accès au groupe `AllUsers` ou `AuthenticatedUsers`. Un bucket public en lecture divulgue les objets à quiconque ; un bucket public en écriture est pire, car un attaquant capable de faire `PutObject` sur un bucket qui sert un site web, un script ou un artefact logiciel contrôle ce que les victimes récupèrent.

## Lire un bucket public

```bash
aws s3 ls s3://<bucket> --no-sign-request
aws s3 sync s3://<bucket> ./loot --no-sign-request
```

## Tester l'écriture publique

```bash
# AllUsers write: anyone can drop an object
echo probe > /tmp/probe.txt
aws s3 cp /tmp/probe.txt s3://<bucket>/probe.txt --no-sign-request
```

Un `PutObject` anonyme réussi signifie que vous pouvez écraser les objets servis par le bucket. Lorsque le bucket sous-tend un site statique ou un script distribué, remplacer un objet constitue une voie d'injection de contenu et d'exécution de code contre ses consommateurs.

## Vérifier les contrôles d'exposition

```bash
aws s3api get-public-access-block --bucket <bucket>
aws s3api get-bucket-acl --bucket <bucket>
```

## Notes d'exploitation

- `AuthenticatedUsers` désigne *n'importe quel* compte AWS, pas le compte du bucket ; un bucket qui l'accorde est public pour tout attaquant détenant une identité quelconque.
- Les buckets publics en écriture qui contiennent du JavaScript, du firmware ou des artefacts d'installation sont de la plus haute valeur : vous passez de l'exposition de données à une injection de contenu de type supply chain sur tous ceux qui les téléchargent.
- Les ACL au niveau objet peuvent différer de la valeur par défaut du bucket, sondez donc individuellement les clés de grande valeur même lorsque le listage du bucket est refusé.

## Outils

- **AWS CLI** (`--no-sign-request`) : lecture et écriture anonymes.
- **s3scanner** : signale les permissions de lecture et d'écriture par bucket.

## Références

- [HackTricks Cloud : accès public S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [Rhino Security Labs : penetration testing AWS storage](https://rhinosecuritylabs.com/aws/penetration-testing-aws-storage/)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [GrayhatWarfare : buckets publics](https://buckets.grayhatwarfare.com/)
- [S3Scanner](https://github.com/sa7mon/S3Scanner)
- [cloud_enum](https://github.com/initstring/cloud_enum)
