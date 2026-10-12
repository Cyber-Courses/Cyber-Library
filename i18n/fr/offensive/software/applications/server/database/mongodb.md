---
title: "MongoDB"
order: 6
description: "La surface offensive de MongoDB atteint en tant que service : exposition non authentifiée par défaut dans les anciens déploiements, énumération des bases de données et des collections, et exécution de JavaScript côté serveur via $where et mapReduce lorsqu'il est activé."
keywords:
  - MongoDB
  - unauthenticated
  - server-side JavaScript
  - $where
  - mapReduce
---

# MongoDB

MongoDB (port 27017) s'exécutait historiquement **sans authentification** et écoutait sur toutes les interfaces, donc les instances exposées restent un point d'ancrage courant d'accès aux données sans authentification. Cette page couvre le serveur atteint en tant que service ; les requêtes contrôlées par l'attaquant depuis une application web relèvent de l'[injection d'opérateur NoSQL](../web/code/injection/database/nosql/mongodb/index.md) et se trouvent sous l'arborescence web.

## Accès et énumération

```bash
# Connect (mongosh, or the legacy mongo shell); no credentials where auth is off
mongosh "mongodb://<target>:27017"
#  show dbs ; use <db> ; show collections ; db.<coll>.find()

# Fingerprint and check for no-auth access
nmap -p27017 --script mongodb-info,mongodb-databases <target>
```

## JavaScript côté serveur

Lorsque le JavaScript côté serveur est activé, `$where` et `mapReduce` évaluent le JavaScript de l'attaquant à l'intérieur du processus de la base de données, ce qui constitue un levier de déni de service et, sur les versions anciennes ou mal configurées, une voie vers l'exécution dans le contexte du moteur :

```javascript
db.coll.find({ $where: "sleep(5000) || true" });   // timing / DoS oracle
db.coll.mapReduce(function(){ /* JS */ }, function(k,v){return v}, {out:"o"});
```

## Notes d'exploitation

- L'exposition sans authentification par défaut est l'élément marquant : énumérez avec `show dbs` et extrayez les collections directement là où l'authentification n'a jamais été activée.
- Même avec l'authentification, vérifiez la présence d'un utilisateur applicatif sur-privilégié et des rôles **`__system`** ou admin atteignables avec des identifiants faibles.
- Le JavaScript côté serveur (`security.javascriptEnabled`) reste **activé** par défaut (déprécié depuis MongoDB 8.0 mais non désactivé par défaut), donc `$where`/`mapReduce` sont généralement disponibles à moins qu'un opérateur ne les ait désactivés.
- MongoDB exposé sans authentification a fait l'objet de rançongiciels de masse dans la nature, donc une instance ouverte constitue à la fois un risque de vol de données et un risque d'intégrité lors d'une mission.

## Outils

- **mongosh** : shell natif pour l'énumération et les requêtes JavaScript.
- **NoSQLMap** : énumération MongoDB automatisée et test d'injection.
- **nmap `mongodb-*`** : empreinte non authentifiée et listage des bases de données.

## Références

- [HackTricks: pentesting MongoDB (27017)](https://hacktricks.wiki/en/network-services-pentesting/27017-27018-mongodb.html)
- [MongoDB: security checklist](https://www.mongodb.com/docs/manual/administration/security-checklist/)
