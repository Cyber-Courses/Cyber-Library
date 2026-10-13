---
title: "Authentification et accès anonyme : atteindre Grafana"
order: 1
description: "Grafana est livré avec admin/admin et, lorsqu'il est configuré, autorise un accès anonyme qui accorde un rôle viewer (ou supérieur) sans connexion. Les identifiants par défaut, les mots de passe faibles et un rôle d'organisation anonyme trop permissif donnent chacun accès aux tableaux de bord et, avec suffisamment de privilèges, aux sources de données et à leurs identifiants stockés."
keywords:
  - grafana admin
  - anonymous access
  - default credentials
  - viewer role
  - org role
---

# Authentification et accès anonyme

L'accès à Grafana provient d'identifiants ou d'un accès anonyme. L'administrateur par défaut est `admin`/`admin`, invité à le changer à la première connexion mais fréquemment laissé tel quel ou défini faiblement. Grafana prend aussi en charge un accès anonyme (`[auth.anonymous] enabled = true`) qui attribue aux visiteurs non authentifiés un rôle d'organisation, prévu comme Viewer mais parfois configuré comme Editor ou Admin, ce qui accorde un accès sans aucune connexion. Atteindre Grafana est donc souvent aussi simple que de s'y rendre (anonyme) ou d'essayer le compte par défaut, et le rôle obtenu détermine l'impact : un Viewer voit les tableaux de bord et les données qu'ils révèlent, tandis qu'un Editor/Admin atteint les sources de données et les fonctionnalités qui exposent leurs identifiants stockés et le proxy SSRF.

```bash
# default admin
curl -sk https://<target>:3000/login -H 'Content-Type: application/json' \
  -d '{"user":"admin","password":"admin"}'
# anonymous access: dashboards/data reachable with no login if enabled
curl -sk https://<target>:3000/api/search                 # lists dashboards if anon-enabled
curl -sk https://<target>:3000/api/org                     # the anonymous org/role
```

## Notes d'exploitation

- Essayez d'abord l'accès anonyme (il ne nécessite aucun identifiant) et le compte par défaut `admin`/`admin` ; les deux sont courants, et un rôle anonyme trop permissif (Editor/Admin) est un accès élevé immédiat.
- Le rôle décide de la portée : Viewer expose les tableaux de bord et les résultats de requêtes (divulgation de données), tandis qu'Editor/Admin atteint la configuration des sources de données et le proxy, voir [SSRF et identifiants des sources de données](data-source-ssrf-and-credentials.md).
- Les mots de passe admin faibles/réutilisés et les clés d'API sont des voies supplémentaires ; une clé d'API Grafana fuitée s'authentifie directement auprès de l'API.
- Là où les identifiants et l'accès anonyme échouent, la [traversée de chemin](path-traversal-file-read.md) non authentifiée peut tout de même lire des secrets serveur sur une version vulnérable.

## Références

- [Grafana: anonymous authentication](https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/)
- [HackTricks: Grafana](https://book.hacktricks.xyz/network-services-pentesting/3000-pentesting-grafana)
