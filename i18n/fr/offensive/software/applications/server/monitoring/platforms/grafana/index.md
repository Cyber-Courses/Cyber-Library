---
title: "Grafana : attaquer les tableaux de bord d'observabilité"
order: 6
description: "Grafana est une plateforme web de tableaux de bord qui se connecte à des sources de données détenant des identifiants pour des bases de données et des services internes. La surface est l'accès par défaut et anonyme, la traversée de chemin de plugin non authentifiée qui lit les fichiers serveur, la server-side request forgery via le proxy des sources de données avec les identifiants de sources de données stockés, et les vulnérabilités connues de plugins et de serveur."
keywords:
  - grafana
  - path traversal
  - data source
  - ssrf
  - dashboards
---

# Grafana

Grafana est une application web largement déployée pour les tableaux de bord et la visualisation, se connectant à des sources de données (Prometheus, Elasticsearch, bases de données SQL, API cloud) pour lesquelles il stocke des identifiants. Elle s'attaque de plusieurs façons. L'accès provient des identifiants par défaut (`admin`/`admin`) et de l'accès anonyme/viewer là où il est activé. Une traversée de chemin de plugin non authentifiée a permis de lire des fichiers arbitraires depuis le serveur Grafana, récupérant sa configuration et ses secrets. Le proxy des sources de données est une primitive de server-side request forgery, atteignant des services internes depuis l'hôte Grafana, et les identifiants de sources de données stockés donnent un accès direct à ces bases de données et services. Et Grafana et ses plugins ont d'autres vulnérabilités connues. Une compromission de Grafana produit donc à la fois un accès au serveur et les identifiants de tout ce qu'il visualise.

```bash
curl -sk https://<target>:3000/login | grep -ioE 'Grafana v[0-9.]+'   # fingerprint (default 3000)
```

## Sous-thèmes

- **[Énumération](enumeration.md)** : identification du produit et de la version.
- **[Authentification et accès anonyme](authentication-and-anonymous-access.md)** : comptes par défaut et visualisation anonyme.
- **[Lecture de fichier par traversée de chemin](path-traversal-file-read.md)** : la traversée de plugin non authentifiée.
- **[SSRF et identifiants des sources de données](data-source-ssrf-and-credentials.md)** : SSRF du proxy et secrets stockés.
- **[Exploits connus](known-exploits.md)** : autres vulnérabilités de plugins et de serveur.

## Références

- [Grafana documentation](https://grafana.com/docs/)
- [Grafana security advisories](https://grafana.com/security/security-advisories/)
