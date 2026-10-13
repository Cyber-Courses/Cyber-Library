---
order: 1
title: "Dépôt public : extraire des secrets d'images ECR exposées"
description: "Récupérer des images depuis des dépôts ECR publics ou faiblement cadrés pour en extraire des secrets et comprendre les services internes."
keywords:
  - ECR
  - dépôt public
  - récupération d'image
  - secrets
  - registre
---

# Dépôt public

Les dépôts ECR peuvent être rendus publics via la galerie ECR Public, ou laissés récupérables par une politique de dépôt trop large. Dans les deux cas, quiconque peut récupérer l'image obtient l'intégralité de son historique de couches, qui contient fréquemment des credentials en dur, des jetons d'API, des noms d'hôtes internes et du code source.

## Récupérer et inspecter

```bash
# Public gallery image
docker pull public.ecr.aws/<alias>/<repo>:<tag>

# Inspect layers and history for secrets
docker history --no-trunc <image>
dive <image>            # interactive layer explorer
```

Analysez les couches décompressées avec un scanner de secrets plutôt qu'à la main :

```bash
docker save <image> -o img.tar && mkdir x && tar -xf img.tar -C x
trufflehog filesystem x/
```

## Notes d'exploitation

- Les secrets présents dans une couche ancienne persistent même si une couche ultérieure supprime le fichier, analysez donc toujours l'historique complet, pas le système de fichiers final.
- Les noms et tags d'images internes révèlent l'inventaire des services et leur version, ce qui amorce le ciblage des charges de travail en cours d'exécution.
- Un dépôt laissé récupérable en inter-comptes relève du cas [politique de dépôt](repository-policy.md) ; un dépôt réellement public ne nécessite aucun credential.

## Outils

- **docker** / **crane** : récupérer et exporter des images.
- **dive** : inspecter les couches de manière interactive.
- **TruffleHog** / **gitleaks** : analyser les couches extraites à la recherche de secrets.

## Références

- [HackTricks Cloud: ECR](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecr-enum.html)
- [TruffleHog (Truffle Security)](https://github.com/trufflesecurity/trufflehog)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
