---
order: 2
title: "Images et registres : attaquer la chaîne d'approvisionnement des conteneurs"
description: "Les images de conteneur sont distribuées via des registres et assemblées à partir d'images de base, deux surfaces d'attaque de la chaîne d'approvisionnement. Un attaquant énumère et tire depuis des registres exposés ou non authentifiés, extrait les secrets intégrés dans les couches d'image, et empoisonne ou backdoore des images de base de sorte que chaque build en aval exécute du code attaquant."
keywords:
  - registre de conteneur
  - image docker
  - chaîne d'approvisionnement
  - couches d'image
  - image de base
---

# Images et registres

Une image est une pile de couches de système de fichiers plus une configuration, et un registre est le serveur qui les stocke et les sert. Les deux sont des cibles de chaîne d'approvisionnement. Un registre atteignable sans authentification, ou avec des identifiants faibles, permet à un attaquant de tirer des images privées (et les secrets qu'elles contiennent) et parfois d'en pousser des malveillantes. Les images elles-mêmes fuient des secrets ajoutés pendant le build et que l'on a cru supprimés plus tard, car chaque couche est conservée. Et une image de base qu'un attaquant peut influencer empoisonne chaque image construite dessus.

Trouver des registres et inspecter des images :

```bash
# registry API v2 is unauthenticated to probe by default
curl -s http://<registry>:5000/v2/_catalog
curl -s http://<registry>:5000/v2/<repo>/tags/list
# pull and inspect an image's build history for secrets
docker pull <registry>:5000/<repo>:<tag>
docker history --no-trunc <registry>:5000/<repo>:<tag>
```

## Sous-thèmes

- **[Registry enumeration](registry-enumeration.md)** : lister les dépôts, les tags et les manifestes.
- **[Registry access](registry-access.md)** : tirer et pousser avec des identifiants trouvés ou faibles.
- **[Unauthenticated registry access](unauthenticated-registry-access.md)** : un registre ouvert sans authentification.
- **[Secrets in image layers](secrets-in-image-layers.md)** : extraire les identifiants intégrés dans les couches.
- **[Base image poisoning](base-image-poisoning.md)** : compromettre une image amont pour atteindre les avals.
- **[Image backdooring](image-backdooring.md)** : modifier une image pour exécuter du code attaquant au lancement.

## Références

- [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec/blob/main/spec.md)
- [Docker Registry HTTP API V2](https://distribution.github.io/distribution/spec/api/)
- [HackTricks: 5000 Docker registry](https://book.hacktricks.xyz/network-services-pentesting/5000-pentesting-docker-registry)
