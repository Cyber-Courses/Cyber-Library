---
title: "Cloud DNS : détourner les enregistrements orphelins pour la prise de contrôle de sous-domaine"
order: 3
description: "Détourner les enregistrements Cloud DNS orphelins qui pointent vers des ressources GCP libérées pour la prise de contrôle de sous-domaine."
keywords:
  - Cloud DNS
  - DNS orphelin
  - prise de contrôle de sous-domaine
  - enregistrement DNS
  - détournement
  - GCP
---

# Cloud DNS

Un enregistrement Cloud DNS orphelin pointe un nom vers une ressource GCP qui n'existe plus : une IP statique globale ou régionale libérée, un bucket de site web GCS démantelé, un mappage de domaine personnalisé App Engine ou Cloud Run qui a été supprimé, ou une adresse d'équilibreur de charge supprimée. Comme le nom résout toujours vers GCP, quiconque recrée une ressource qui attire la même adresse ou le même identifiant capture chaque requête vers ce sous-domaine, ce qui procure un point d'appui d'origine de confiance pour le phishing, le vol de cookies et l'émission de certificats.

## Trouver les enregistrements orphelins

```bash
# every managed zone and its records
gcloud dns managed-zones list
gcloud dns record-sets list --zone <zone> \
  --format="table(name,type,rrdatas.list())"
# resolve each A / CNAME target and flag released IPs, NXDOMAIN, or NoSuchBucket
```

## Revendiquer la ressource sous-jacente

- **IP statique libérée** : un enregistrement A vers une adresse globale/régionale libérée est capturé en allouant des adresses (`gcloud compute addresses create`) jusqu'à ce que GCP réémette la même.
- **Site web GCS** : un CNAME vers `c.storage.googleapis.com` pour un bucket supprimé est récupéré en créant un bucket portant exactement le même nom.
- **App Engine / Cloud Run / GCLB** : un mappage de domaine personnalisé vers un service supprimé est récupéré en mappant le même nom d'hôte vers un service que vous contrôlez dans votre propre projet.

## Notes d'exploitation

- Un sous-domaine capturé hérite de la confiance du domaine parent : les cookies cadrés sur le parent, les listes d'autorisation de redirection SSO et les attentes des utilisateurs sont tous transférés.
- Le contrôle du nom valide les défis ACME HTTP-01 et DNS-01, vous pouvez donc générer un certificat valide pour le sous-domaine.
- Les `rrdatas` d'un enregistrement A qui ne répond plus, ou un CNAME vers un point de terminaison GCP non revendiqué, constituent l'empreinte à rechercher à grande échelle.

## Outils

- **gcloud** (`dns record-sets list`) : inventaire des enregistrements.
- **nuclei** (modèles `takeovers`), **subjack** / **tko-subs** : détecter les empreintes revendicables à grande échelle.
- **can-i-take-over-xyz** : la référence des empreintes indiquant quels fournisseurs sont revendicables.

## Références

- [can-i-take-over-xyz (empreintes de prise de contrôle de sous-domaine)](https://github.com/EdOverflow/can-i-take-over-xyz)
- [HackTricks Cloud : GCP DNS](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
