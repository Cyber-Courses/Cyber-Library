---
title: "cAdvisor et métriques : fuites d'environnement et de topologie depuis les points de terminaison de statistiques"
order: 6
description: "cAdvisor et les divers points de terminaison de métriques exposent des statistiques et des métadonnées par conteneur. Lorsqu'ils sont accessibles sans authentification, ils divulguent des noms de conteneurs, des images, des libellés, et parfois des variables d'environnement et des lignes de commande, donnant à un attaquant une carte des charges de travail du cluster et occasionnellement des secrets passés comme arguments ou environnement de conteneur."
keywords:
  - cadvisor
  - métriques
  - port 4194
  - statistiques de conteneur
  - divulgation d'informations
---

# cAdvisor et métriques

cAdvisor collecte les statistiques de ressources de chaque conteneur et les servait historiquement sur le port 4194 ; le kubelet expose également les points de terminaison `/metrics`, `/metrics/cadvisor` et `/stats`, et les clusters exécutent metrics-server et des exportateurs Prometheus. Ce sont des surfaces de surveillance, mais lorsqu'elles sont accessibles sans authentification, elles divulguent une carte détaillée de ce qui s'exécute et où : noms de conteneurs, images, libellés de pods et d'espaces de noms, et utilisation des ressources. Certaines exposent les lignes de commande et l'environnement des conteneurs, qui peuvent inclure des secrets passés comme arguments ou variables.

Sondez les points de terminaison :

```bash
# legacy cAdvisor UI/API
curl -s http://<node>:4194/api/v1.3/containers/ | python3 -m json.tool | head -60
# kubelet-exposed cAdvisor metrics (via read-only port or the kubelet)
curl -s http://<node>:10255/metrics/cadvisor | head
curl -s http://<node>:10255/stats/summary | python3 -m json.tool | head
```

## Ce qui fuit

```bash
# container specs include labels, images, and sometimes env/args
curl -s http://<node>:4194/api/v1.3/containers/ | \
  grep -iE 'image|namespace|env|argv|TOKEN|SECRET|PASSWORD'
# the topology alone maps every workload, aiding target selection
```

## Notes d'exploitation

- Traitez-les comme de la reconnaissance : ils donnent rarement une exécution, mais ils révèlent l'inventaire complet des charges de travail (images, espaces de noms, libellés), ce qui guide l'orientation des voies RBAC et d'évasion de pod.
- Les variables d'environnement ou les arguments de ligne de commande apparaissant dans les métadonnées de conteneur contiennent occasionnellement des identifiants passés de manière non sûre ; recherchez dans les sorties les chaînes ressemblant à des secrets.
- Ces points de terminaison sont souvent laissés ouverts sur le réseau du nœud même lorsque les API principales sont verrouillées, de sorte qu'ils constituent un premier aperçu utile et peu contraignant d'un cluster par ailleurs durci.

## Références

- [cAdvisor](https://github.com/google/cadvisor)
- [Kubernetes : points de terminaison de métriques du kubelet](https://kubernetes.io/docs/reference/instrumentation/metrics/)
- [kube-hunter : cAdvisor](https://aquasecurity.github.io/kube-hunter/)
