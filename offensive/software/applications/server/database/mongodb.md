---
title: "MongoDB"
description: "The offensive surface of MongoDB reached as a service: unauthenticated exposure by default in older deployments, database and collection enumeration, and server-side JavaScript execution through $where and mapReduce where it is enabled."
keywords:
  - MongoDB
  - unauthenticated
  - server-side JavaScript
  - $where
  - mapReduce
---

# MongoDB

MongoDB (port 27017) historically ran **without authentication** and bound to all interfaces, so exposed instances remain a common unauthenticated data-access foothold. This page covers the server reached as a service; attacker-controlled queries from a web application are [NoSQL operator injection](../web/code/injection/database/nosql/mongodb/index.md) and live under the web tree.

## Access and enumeration

```bash
# Connect (mongosh, or the legacy mongo shell); no credentials where auth is off
mongosh "mongodb://<target>:27017"
#  show dbs ; use <db> ; show collections ; db.<coll>.find()

# Fingerprint and check for no-auth access
nmap -p27017 --script mongodb-info,mongodb-databases <target>
```

## Server-side JavaScript

Where server-side JavaScript is enabled, `$where` and `mapReduce` evaluate attacker JavaScript inside the database process, which is a denial-of-service lever and, on older/misconfigured builds, a path toward execution in the engine context:

```javascript
db.coll.find({ $where: "sleep(5000) || true" });   // timing / DoS oracle
db.coll.mapReduce(function(){ /* JS */ }, function(k,v){return v}, {out:"o"});
```

## Exploitation notes

- The default-no-auth exposure is the headline: enumerate `show dbs` and dump collections directly where authentication was never enabled.
- Even with auth, check for an over-privileged application user and for **`__system`** or admin roles reachable with weak credentials.
- Server-side JavaScript (`security.javascriptEnabled`) still defaults to **enabled** (deprecated as of MongoDB 8.0 but not disabled by default), so `$where`/`mapReduce` are usually available unless an operator turned them off.
- MongoDB exposed with no auth has been mass-ransomed in the wild, so an open instance is both a data-theft and an integrity concern on an engagement.

## Tools

- **mongosh**: native shell for enumeration and JavaScript queries.
- **NoSQLMap**: automated MongoDB enumeration and injection testing.
- **nmap `mongodb-*`**: unauthenticated fingerprinting and database listing.

## References

- [HackTricks: pentesting MongoDB (27017)](https://hacktricks.wiki/en/network-services-pentesting/27017-27018-mongodb.html)
- [MongoDB: security checklist](https://www.mongodb.com/docs/manual/administration/security-checklist/)
