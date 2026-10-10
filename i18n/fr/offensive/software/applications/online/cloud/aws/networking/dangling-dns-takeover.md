---
title: "Prise de contrôle de DNS orphelin : revendiquer la ressource sous-jacente d'un enregistrement orphelin"
order: 5
description: "Revendiquer la ressource sous-jacente d'un enregistrement DNS orphelin (origine S3, CloudFront ou ELB) pour prendre le contrôle du sous-domaine."
keywords:
  - dangling DNS
  - subdomain takeover
  - CNAME
  - S3
  - CloudFront
---

# Prise de contrôle de DNS orphelin

Un enregistrement orphelin pointe un nom vers une ressource AWS qui n'existe plus : un bucket de site web S3 supprimé, une Elastic IP libérée, une distribution CloudFront démantelée, ou un ELB retiré. Parce que le nom résout toujours vers le fournisseur, quiconque peut recréer une ressource portant le même identifiant capture chaque requête vers ce sous-domaine, ce qui donne un point d'appui d'origine de confiance pour le phishing, le vol de cookies et l'émission de certificats.

## Trouver les enregistrements orphelins

```bash
# records whose alias/CNAME targets an AWS resource
aws route53 list-resource-record-sets --hosted-zone-id <zone-id> \
  --query "ResourceRecordSets[?Type=='CNAME' || AliasTarget].[Name,Type,ResourceRecords[0].Value,AliasTarget.DNSName]"
# then resolve each target and flag NXDOMAIN / NoSuchBucket / unclaimed endpoints
```

## Revendiquer la ressource sous-jacente

- **Origine de site web S3** : la cible `<name>.s3-website-<region>.amazonaws.com` renvoyant `NoSuchBucket` signifie que vous créez un bucket portant ce nom exact dans cette région et que vous servez votre contenu.
- **CloudFront** : un CNAME vers le `*.cloudfront.net` d'une distribution retirée peut être revendiqué en enregistrant le nom de domaine alternatif sur une distribution que vous possédez.
- **Elastic IP / ELB** : un enregistrement A vers une EIP libérée se capture en réallouant des EIP jusqu'à obtenir la même adresse ; un nom DNS d'ELB mort se récupère en créant un load balancer auquel il est attribué.

## Notes d'exploitation

- Un sous-domaine capturé hérite de la confiance du domaine parent : les cookies de session scopés à `.example.com`, les listes d'autorisation de redirection SSO et l'attente des utilisateurs sont tous transférés.
- Le contrôle du nom franchit aussi les défis ACME HTTP-01 et DNS-01, vous pouvez donc générer un certificat valide pour le sous-domaine.
- Croisez avec l'[énumération S3](../storage/s3/enumeration.md) pour découvrir le cas de l'origine bucket et [Route53](route53.md) pour l'inventaire des enregistrements.

## Outils

- **nuclei** (templates `takeovers`) et **subjack** / **tko-subs** : détecter les empreintes revendicables à grande échelle.
- **AWS CLI** : recréer le bucket, l'alias de distribution ou l'EIP pour revendiquer la cible.

## Références

- [HackTricks Cloud : prise de contrôle de sous-domaine](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-route53-enum.html)
- [Can I take over XYZ (empreintes de prise de contrôle de second ordre)](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
- [fwd:cloudsec : conférences sur la sécurité cloud](https://fwdcloudsec.org/)
