---
title: "Énumération : découvrir les buckets S3 et leurs objets"
order: 1
description: "Découverte des buckets et objets S3 par devinette de noms, DNS et sondage des permissions, puis listage du contenu lisible."
keywords:
  - S3 enumeration
  - bucket
  - ListBucket
  - object
  - discovery
---

# Énumération

Les buckets S3 vivent dans un espace de noms global unique, ils peuvent donc être trouvés sans aucune identité : un nom se résout ou non. L'énumération consiste à deviner des noms à partir de l'image de marque de la cible, à les confirmer via DNS ou HTTP, puis à sonder ce que l'identité courante (ou un appelant anonyme) peut lister et lire.

## Découverte non authentifiée

```bash
# a bucket name resolves in DNS if it exists
host target-backups.s3.amazonaws.com

# HTTP status distinguishes exists-but-denied (403) from absent (404)
curl -s -o /dev/null -w '%{http_code}\n' https://target-backups.s3.amazonaws.com

# anonymous list, if the bucket allows it
aws s3 ls s3://target-backups --no-sign-request
```

## Deviner des noms à grande échelle

```bash
# permutation + wordlist scanning
s3scanner scan --bucket-file names.txt
cloud_enum -k target -k target-prod            # S3 + GCP + Azure in one pass
bucket-finder names.txt                        # classic read/list prober
# GrayhatWarfare: search already-indexed public buckets and their files by keyword (web UI/API)

# resolve a bucket's region from the endpoint redirect
dig +short <bucket>.s3.amazonaws.com
curl -sI https://<bucket>.s3.amazonaws.com | grep -i x-amz-bucket-region
```

## Listage authentifié

```bash
aws s3api list-buckets --query 'Buckets[].Name'
aws s3 ls s3://<bucket> --recursive
aws s3api list-objects-v2 --bucket <bucket> --query 'Contents[].Key'
```

## Récupérer les objets supprimés et écrasés

Un bucket versionné conserve chaque objet antérieur et supprimé ; `ListBucket` les masque, mais `s3:ListBucketVersions` et `GetObjectVersion` les ramènent, ce qui fait souvent réapparaître des secrets qui avaient été « supprimés » :

```bash
aws s3api list-object-versions --bucket <bucket> \
  --query '{v:Versions[].{k:Key,id:VersionId},d:DeleteMarkers[].Key}'
# fetch a specific old or delete-marked version
aws s3api get-object --bucket <bucket> --key <key> --version-id <VersionId> out.bin
```

## Notes d'exploitation

- Un `403` est un résultat positif : le bucket existe et mérite d'être attaqué via sa policy ou ses ACL, même lorsque le listage est refusé.
- Les clés d'objets révèlent la structure : les préfixes de sauvegardes datées, `terraform.tfstate` et les clés `.env` sont des cibles de grande valeur à demander directement, même sans `ListBucket`.
- La région compte pour l'endpoint ; une redirection dans la réponse nomme la région du bucket.

## Outils

- **s3scanner** : analyse de l'existence des buckets et des permissions à partir d'une liste de noms.
- **cloud_enum** : découverte de ressources publiques multi-fournisseurs.
- **bucket-finder** : sondage de lecture et de listage sur une wordlist de noms.
- **GrayhatWarfare** : index interrogeable des buckets et fichiers déjà publics.
- **AWS CLI** (`--no-sign-request`, `list-object-versions`) : listage/récupération anonyme et récupération de versions.
- **Pacu** (`s3__bucket_finder`, `s3__download_bucket`) : trouver et récupérer les buckets lisibles sur l'ensemble du compte.

## Références

- [s3scanner](https://github.com/sa7mon/S3Scanner)
- [cloud_enum (initstring)](https://github.com/initstring/cloud_enum)
- [GrayhatWarfare : buckets publics](https://buckets.grayhatwarfare.com/)
- [HackTricks Cloud : énumération S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
