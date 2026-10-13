---
title: "VPC : pivoter à travers les réseaux, le peering et le VPC partagé"
order: 2
description: "Pivoter à travers les réseaux VPC, le peering et le VPC partagé pour atteindre des instances et services internes."
keywords:
  - VPC
  - peering
  - VPC partagé
  - interne
  - pivot
  - réseau
---

# VPC

Un VPC GCP est un réseau global de sous-réseaux régionaux. Depuis un point d'appui sur une instance, l'ensemble accessible est défini par le VPC, ses **peerings**, tout rattachement de **VPC partagé** et ses routes. Énumérer cette topologie transforme un seul hôte compromis en une carte de services internes vers lesquels pivoter.

## Cartographier le réseau accessible

```bash
gcloud compute networks list
gcloud compute networks subnets list --format="table(name,region,network,ipCidrRange)"

# peerings extend reach into other VPCs (often other projects)
gcloud compute networks peerings list --network <vpc>

# shared VPC: a host project lends subnets to service projects
gcloud compute shared-vpc get-host-project <project>
gcloud compute shared-vpc associated-projects list <host-project>
```

## Atteindre les hôtes internes

```bash
# internal addresses of instances you can now route to
gcloud compute instances list \
  --format="table(name,networkInterfaces[].networkIP.list(),networkInterfaces[].network.list())"
```

Avec une instance servant de point d'appui, utilisez-la comme pivot (transfert SSH local/dynamique, ou un implant) pour atteindre les services internes uniquement à travers les réseaux peerés et partagés.

## Notes d'exploitation

- Le **peering** VPC est non transitif mais s'enchaîne souvent en pratique à travers un réseau hub ; suivez le graphe de peering pour trouver l'instance à la plus large portée.
- Le **VPC partagé** signifie qu'une instance de projet de service se trouve sur le sous-réseau du projet hôte, de sorte que compromettre un projet de service peut exposer la plage interne partagée des autres.
- Private Google Access et Cloud NAT donnent aux instances un trafic sortant même sans IP publique, ce qui compte pour votre chemin d'exfiltration et de rappel.

## Outils

- **gcloud** (`compute networks`, `subnets`, `peerings`) : énumération de la topologie.
- **SSH / proxychains** : pivoter à travers une instance servant de point d'appui vers les plages internes.

## Références

- [HackTricks Cloud : GCP VPC](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google Cloud : peering de réseau VPC](https://cloud.google.com/vpc/docs/vpc-peering)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
