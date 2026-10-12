---
title: "Global Accelerator : mettre des endpoints en frontal via listeners et endpoint groups"
order: 6
description: "Détourner les listeners et endpoint groups de Global Accelerator pour atteindre des endpoints internes ou les mettre en frontal."
keywords:
  - Global Accelerator
  - listener
  - endpoint group
  - anycast
  - routing
---

# Global Accelerator

Global Accelerator place une paire d'IP anycast statiques devant des endpoints backend (ALB, NLB, EC2, Elastic IP) et y achemine le trafic via le backbone AWS. Avec un accès en écriture, vous ajoutez ou repointez des listeners et des endpoint groups pour mettre en frontal un endpoint que vous contrôlez, ou pour exposer un backend interne via les adresses publiques de l'accélérateur.

## Énumérer les accélérateurs et les backends

```bash
aws globalaccelerator list-accelerators
aws globalaccelerator list-listeners --accelerator-arn <arn>
aws globalaccelerator list-endpoint-groups --listener-arn <listener-arn> \
  --query "EndpointGroups[].EndpointDescriptions[].EndpointId"
```

## Notes d'exploitation

- Ajouter un endpoint à un groupe existant peut placer un backend que vous contrôlez derrière une IP d'accélérateur de confiance et autorisée par liste d'autorisation.
- Les IP anycast statiques sont fréquemment inscrites sur des listes d'autorisation par les partenaires ; mettre votre endpoint en frontal derrière elles hérite de cette confiance.
- Les endpoint groups peuvent référencer des backends dans d'autres régions, élargissant la portée depuis un seul accélérateur.

## Outils

- **AWS CLI** (`globalaccelerator ...`) : énumération et modifications de listeners/endpoints.
- **ScoutSuite** / **Prowler** : inventaire des accélérateurs et de leurs endpoints.

## Références

- [AWS : composants de Global Accelerator](https://docs.aws.amazon.com/global-accelerator/latest/dg/introduction-components.html)
- [HackTricks Cloud : services réseau AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [CloudFox : cartographier les chemins réseau cloud accessibles](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
