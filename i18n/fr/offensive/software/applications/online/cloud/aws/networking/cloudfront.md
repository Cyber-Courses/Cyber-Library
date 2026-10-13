---
title: "CloudFront : abus de l'origine et du comportement de cache"
order: 4
description: "Abuser des distributions, origines et comportements de cache CloudFront pour contourner des contrôles ou atteindre des origines protégées."
keywords:
  - CloudFront
  - distribution
  - origin
  - cache
  - CDN
---

# CloudFront

CloudFront se place devant des origines (buckets S3, ALB, hôtes personnalisés) et applique la mise en cache et le traitement des requêtes. Sa configuration divulgue les adresses d'origine que l'edge est censé masquer, et un accès d'origine faible ou un comportement sur les en-têtes transférés vous permet d'atteindre l'origine directement ou d'empoisonner ce que d'autres utilisateurs reçoivent. Avec `cloudfront:Get*`/`List*`, vous lisez les origines et comportements de chaque distribution.

## Lire les distributions et les origines

```bash
aws cloudfront list-distributions \
  --query "DistributionList.Items[].[Id,DomainName,Origins.Items[].DomainName]"
aws cloudfront get-distribution-config --id <dist-id>
```

## Atteindre l'origine directement

```bash
# origin domains revealed above are often reachable without the CDN in front,
# bypassing WAF/geo/signed-URL controls enforced only at the edge
curl -H 'Host: app.example.com' https://<origin-domain-from-config>/
```

## Notes d'exploitation

- Les contrôles appliqués uniquement à l'edge (WAF, URL signées, restriction géographique) sont nuls si l'origine accepte les requêtes directes ; le domaine d'origine issu de la configuration est le contournement.
- Une origine S3 sans origin access control peut être lisible ou revendicable directement ; une origine manquante est un candidat à la [prise de contrôle de DNS orphelin](dangling-dns-takeover.md).
- Les en-têtes que CloudFront transfère et sur lesquels il indexe le cache déterminent si une charge utile d'empoisonnement de cache ou d'en-tête Host atteint et persiste à l'origine.

## Outils

- **AWS CLI** (`cloudfront get-distribution-config`) : divulgation de l'origine et du comportement.
- **Burp Suite** : test d'empoisonnement de cache et d'en-tête Host contre l'edge.

## Références

- [HackTricks Cloud : AWS CloudFront](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cloudfront-enum.html)
- [AWS : restreindre l'accès à une origine](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-origin.html)
- [can-i-take-over-xyz : empreintes de prise de contrôle CloudFront et de sous-domaine](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
