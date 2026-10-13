---
title: "Équilibrage de charge : atteindre les backends via les règles de transfert"
order: 4
description: "Cartographier et atteindre les backends via les équilibreurs de charge GCP, les règles de transfert exposées et les services backend."
keywords:
  - équilibrage de charge
  - règle de transfert
  - service backend
  - exposition
  - GCP
  - trafic entrant
---

# Équilibrage de charge

Un équilibreur de charge GCP est assemblé à partir de règles de transfert, de proxys cibles, de mappages d'URL et de services backend qui pointent vers des groupes d'instances, des groupes de points de terminaison réseau ou des buckets. Leur énumération révèle les frontends publics et les backends qu'ils atteignent, et des backends mal configurés ou un accès direct à l'origine vous permettent de contourner les contrôles en façade.

## Cartographier les frontends vers les backends

```bash
# public entry points
gcloud compute forwarding-rules list \
  --format="table(name,IPAddress,IPProtocol,portRange,target)"

# what each backend service points at, and its security policy
gcloud compute backend-services list \
  --format="table(name,protocol,backends[].group.list(),securityPolicy)"
gcloud compute url-maps list
```

## Notes d'exploitation

- Un service backend sans **politique de sécurité** Cloud Armor attachée est accessible sans filtrage WAF ; une politique attachée uniquement au niveau de l'équilibreur de charge est contournée en atteignant directement l'IP propre de l'instance backend (associez à [règles de pare-feu](firewall-rules.md)).
- Les équilibreurs de charge internes n'exposent les services qu'à l'intérieur du VPC, ils deviennent donc accessibles après un pivot [VPC](vpc.md), et non depuis internet.
- Un bucket backend (`gcloud compute backend-buckets list`) expose un bucket GCS via l'équilibreur de charge ; le bucket sous-jacent peut aussi être accessible directement, voir [Cloud Storage](../storage/cloud-storage/index.md).

## Outils

- **gcloud** (`compute forwarding-rules`, `backend-services`, `url-maps`) : énumération des frontends et backends.
- **nmap** / **curl** : sonder les IP des règles de transfert et tester l'accessibilité directe à l'origine.

## Références

- [HackTricks Cloud : équilibrage de charge GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google Cloud : présentation de l'équilibrage de charge cloud](https://cloud.google.com/load-balancing/docs/load-balancing-overview)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
