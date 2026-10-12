---
title: "Network Firewall : repérer les lacunes de règles et désactiver l'inspection"
order: 9
description: "Énumérer et détourner les rule groups et policies d'AWS Network Firewall pour trouver des lacunes ou désactiver l'inspection."
keywords:
  - Network Firewall
  - rule group
  - firewall policy
  - inspection
  - bypass
---

# Network Firewall

AWS Network Firewall inspecte le trafic du VPC face à des rule groups stateless et stateful liés à une firewall policy. Lire ces règles montre exactement ce qui est filtré et ce qui ne l'est pas, ce qui vous permet d'acheminer l'exfiltration ou le command-and-control via une destination autorisée ; un accès en écriture permet d'affaiblir la policy ou d'abandonner complètement l'inspection.

## Énumérer les règles et la policy

```bash
aws network-firewall list-firewalls
aws network-firewall describe-firewall --firewall-name <name>
aws network-firewall describe-firewall-policy --firewall-policy-name <policy>
aws network-firewall describe-rule-group --rule-group-name <rg> --type STATEFUL
```

## Notes d'exploitation

- L'action par défaut pour le trafic non inspecté ainsi que tout FQDN ou CIDR inscrit sur une liste d'autorisation sont les lacunes : envoyez l'exfiltration vers une destination autorisée plutôt que de lutter contre les règles.
- Un rule group stateful évalué dans le mauvais ordre, ou une règle de TLS SNI manquante, laisse passer du trafic que l'opérateur croit bloqué.
- Avec un accès en écriture, dissocier un rule group ou régler l'action par défaut de la policy sur allow supprime l'inspection sans supprimer le firewall, ce qui paraît anodin dans un inventaire.

## Outils

- **AWS CLI** (`network-firewall describe-*`) : divulgation des rule groups et policies.
- **ScoutSuite** / **Prowler** : inventaire des firewalls et de leurs actions par défaut.

## Références

- [AWS : rule groups de Network Firewall](https://docs.aws.amazon.com/network-firewall/latest/developerguide/rule-groups.html)
- [HackTricks Cloud : AWS Network Firewall](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [CloudFox : cartographier les chemins réseau cloud accessibles](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
