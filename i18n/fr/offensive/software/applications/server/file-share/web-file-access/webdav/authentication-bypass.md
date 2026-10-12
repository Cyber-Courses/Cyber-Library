---
title: "Contournement d'authentification : atteindre les verbes WebDAV malgré une authentification faible"
order: 1
description: "L'authentification WebDAV est fréquemment incohérente : certains verbes sont protégés tandis que d'autres ne le sont pas, la falsification de verbe HTTP atteint un chemin protégé via une méthode non vérifiée, et des identifiants faibles ou par défaut gardent le reste. Contourner ce contrôle d'accès ou le deviner expose les verbes d'écriture qui mènent au dépôt de fichiers et à l'exécution de code."
keywords:
  - webdav auth
  - verb tampering
  - http method
  - default credentials
  - access control
---

# Contournement d'authentification

Le contrôle d'accès WebDAV est souvent appliqué de manière inégale, et cette incohérence constitue le contournement. Une faille courante est la protection restreinte à certaines méthodes : un serveur configuré pour exiger une authentification sur `GET`/`POST` peut laisser `PUT`, `MOVE` ou `PROPFIND` non protégés, de sorte qu'un attaquant atteint une ressource protégée via un verbe non vérifié (falsification de verbe HTTP). Là où l'authentification est imposée, il s'agit fréquemment d'une authentification Basic avec des identifiants faibles ou par défaut qui cèdent au spraying. Passer au-delà expose les verbes d'écriture qui rendent WebDAV dangereux.

```bash
# verb tampering: a protected path reachable via an unprotected method
curl -s -X GET http://<target>/protected/        # 401
curl -s -X PROPFIND http://<target>/protected/ -H 'Depth:1' --data ''   # 207 => PROPFIND unguarded
curl -s -X PUT http://<target>/protected/x.txt --data 'test' -i        # does PUT skip auth?
# weak/default Basic auth
curl -s -u admin:admin -X OPTIONS http://<target>/ -i | grep -i allow
hydra -L users.txt -P pass.txt http-get://<target>/protected/
```

## Notes d'exploitation

- Testez chaque verbe contre un chemin protégé indépendamment : un 401 sur `GET` avec un 207 sur `PROPFIND` ou un succès sur `PUT` révèle une protection restreinte à certaines méthodes, le classique contournement WebDAV par falsification de verbe.
- Les mauvaises configurations WebDAV d'IIS et d'Apache ont historiquement permis exactement cette application inégale ; sondez toujours `PUT`/`MOVE`/`PROPFIND` même lorsque `GET` est verrouillé.
- Là où l'authentification est réellement imposée sur tous les verbes, repliez-vous sur les attaques d'identifiants (authentification Basic par défaut/faible) ou sur des identifiants trouvés ailleurs.
- Un contournement réussi qui atteint `PUT`/`MOVE` mène tout droit à [Téléversement PUT vers RCE](put-upload-to-rce.md) ; un contournement qui n'atteint que `PROPFIND`/`GET` permet tout de même le listage et la traversée.

## Références

- [HackTricks : WebDAV et falsification de verbe](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
- [OWASP : tester les méthodes HTTP](https://owasp.org/www-project-web-security-testing-guide/)
