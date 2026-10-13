---
title: "RDS"
order: 1
description: "Attaquer RDS et Aurora : exposer les données via des snapshots partagés ou publics et atteindre la base de données via des contrôles réseau et d'authentification faibles."
keywords:
  - RDS
  - Aurora
  - snapshot
  - base de données
  - authentification IAM
---

# RDS

RDS et Aurora conservent des données relationnelles derrière deux surfaces séparables. Le **plan de contrôle** permet à un principal doté de permissions RDS de copier ou de partager un **snapshot**, de le restaurer dans une instance qu'il possède, et de lire toute la base de données hors ligne sans jamais toucher à l'originale. Le **plan de données** est l'instance en service, joignable lorsque son point de terminaison est public, que son groupe de sécurité est laxiste, ou que ses identifiants sont faibles ou récupérables.

## Ce qui est regroupé ici

- **[Snapshots](snapshots.md)** : partager, copier et restaurer des snapshots pour lire la base de données hors ligne.
- **[Accès](access.md)** : atteindre des instances en service via des points de terminaison exposés, des identifiants faibles ou l'authentification IAM RDS.

## Références

- [HackTricks Cloud: AWS RDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: sharing a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html)
- [Pacu: AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
