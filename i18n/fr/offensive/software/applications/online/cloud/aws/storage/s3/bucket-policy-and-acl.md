---
title: "Politique de bucket et ACL : lecture, écriture et prise de contrôle entre comptes"
order: 3
description: "Abuser de politiques de bucket trop permissives et d'ACL d'objets pour lire, écrire ou prendre le contrôle du contenu d'un bucket entre comptes."
keywords:
  - bucket policy
  - ACL
  - cross-account
  - PutObject
  - grant
---

# Politique de bucket et ACL

Au-delà de l'interrupteur public/privé, la **politique de ressource** d'un bucket et ses **ACL** accordent à des principaux précis des actions précises, et les deux sont couramment rédigées de façon trop large : une politique qui désigne un compte entier comme principal, ou un `Principal: "*"` avec une condition faible, livre l'accès aux attaquants qui satisfont la condition. Si vous pouvez lire la politique, vous apprenez qui peut atteindre le bucket ; si vous pouvez l'*écrire* (`s3:PutBucketPolicy`), vous vous accordez ce qui vous manque.

## Lire les octrois

```bash
aws s3api get-bucket-policy --bucket <bucket> --query Policy --output text | jq .
aws s3api get-bucket-acl --bucket <bucket>
aws s3api get-object-acl --bucket <bucket> --key <key>
```

## Réécrire une politique que vous pouvez définir

```bash
cat > p.json <<'EOF'
{"Version":"2012-10-17","Statement":[{"Effect":"Allow",
  "Principal":{"AWS":"arn:aws:iam::<your-acct>:root"},
  "Action":"s3:*","Resource":["arn:aws:s3:::<bucket>","arn:aws:s3:::<bucket>/*"]}]}
EOF
aws s3api put-bucket-policy --bucket <bucket> --policy file://p.json
```

## Octrois entre comptes à rechercher

- Une instruction dont le `Principal.AWS` est le root d'un autre compte : tout principal de ce compte disposant de l'action S3 correspondante y accède.
- `s3:PutObject` accordé largement sur un bucket consommé par un pipeline : écrivez une entrée empoisonnée et laissez la victime la traiter.
- Des ACL d'objets accordant `AllUsers`/`AuthenticatedUsers` même lorsque la politique de bucket paraît stricte.

## Notes d'exploitation

- Les contrôles de propriété du bucket et le paramètre de propriétaire d'objet changent le fait qu'une ACL d'un objet téléversé soit honorée ou non ; lorsque `BucketOwnerEnforced` est désactivé, un rédacteur d'un autre compte peut définir des ACL sur les objets qu'il dépose.
- `s3:PutBucketPolicy` sur un bucket que vous ne possédez pas est en soi une primitive de prise de contrôle : réécrivez la politique, puis lisez ou écrasez tout.
- Associez à l'[énumération](enumeration.md) pour trouver les buckets et à l'[accès public](public-access.md) pour le cas anonyme.

## Outils

- **AWS CLI** (`s3api get-/put-bucket-policy`, `get-object-acl`) : lire et réécrire les octrois.
- **Pacu** (`s3__bucket_finder`) : localiser et évaluer les buckets dans une session.

## Références

- [HackTricks Cloud : abus de politique de bucket et d'ACL S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [AWS : évaluation des politiques de bucket et des ACL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-policy-language-overview.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [GrayhatWarfare : buckets publics](https://buckets.grayhatwarfare.com/)
- [S3Scanner](https://github.com/sa7mon/S3Scanner)
- [cloud_enum](https://github.com/initstring/cloud_enum)
