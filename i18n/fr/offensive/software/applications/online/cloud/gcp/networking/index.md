---
title: "Réseau GCP"
order: 7
description: "Attaquer le réseau GCP : exposition par règles de pare-feu, portée VPC, Cloud DNS et prise de contrôle de sous-domaine, et exposition par équilibreur de charge."
keywords:
  - règles de pare-feu
  - VPC
  - Cloud DNS
  - équilibrage de charge
  - prise de contrôle de sous-domaine
  - réseau
---

# Réseau

Le réseau GCP décide de ce qui est accessible : les **règles de pare-feu** qui ouvrent le trafic entrant vers internet, le peering **VPC** et le VPC partagé qui permettent à un point d'appui dans un réseau d'en atteindre un autre, les enregistrements **Cloud DNS** laissés orphelins pointant vers des ressources libérées, et les **équilibreurs de charge** qui exposent les backends. Le réseau compromet rarement un projet à lui seul, mais il transforme un point d'appui accessible en portée à travers l'environnement et capture des noms de confiance.

## Ce qui est regroupé ici

- **[Règles de pare-feu](firewall-rules.md)** : ouvrir le trafic entrant avec `compute.firewalls.create`/`update`, et trouver les règles déjà ouvertes à `0.0.0.0/0`.
- **[VPC](vpc.md)** : peering, VPC partagé et routes pour pivoter vers des instances et services internes.
- **[Cloud DNS](cloud-dns.md)** : détourner les enregistrements orphelins qui pointent vers des ressources GCP libérées.
- **[Équilibrage de charge](load-balancing.md)** : règles de transfert et services backend qui exposent ou atteignent les backends.

Le mouvement basé sur l'identité (usurpation d'identité et `actAs` entre projets) réside dans [identity](../identity/index.md) ; cette surface est la couche réseau.

## Références

- [HackTricks Cloud : réseau GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google Cloud : règles de pare-feu VPC](https://cloud.google.com/firewall/docs/firewalls)
