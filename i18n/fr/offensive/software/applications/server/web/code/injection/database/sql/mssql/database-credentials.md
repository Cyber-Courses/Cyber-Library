---
order: 3
title: "Dumper les hachages de mots de passe des logins SQL Server"
description: "Extraire les hachages de mots de passe des logins SQL Server depuis sys.sql_logins pour un cassage hors ligne, et les sources plus anciennes syslogins et sysxlogins."
keywords:
  - dump de hachages de mots de passe
  - sys.sql_logins
  - password_hash
  - syslogins
  - LOGINPROPERTY
---

# Identifiants de base de données

SQL Server stocke les hachages des logins authentifiés par SQL, et une injection à privilège élevé peut les lire pour un cassage hors ligne, ce qui peut récupérer le mot de passe `sa` ou des identifiants réutilisés ailleurs.

Sur SQL Server 2012 et plus, les hachages sont dans `sys.sql_logins.password_hash` (un `varbinary`), convertis en hexadécimal pour le transport :

```sql
' UNION SELECT name,master.dbo.fn_varbintohexstr(password_hash),NULL FROM sys.sql_logins-- 
```

La chaîne hexadécimale (commençant par `0x0200...` pour le schéma de hachage actuel) est cassée hors ligne, par exemple avec le mode SQL Server 2012/2014 de hashcat. Sur les serveurs plus anciens, les sources diffèrent : `sys.syslogins` et `master..syslogins` exposent une colonne `password` sur 2005/2008, et `master..sysxlogins` sur SQL Server 2000. `LOGINPROPERTY(name,'PasswordHash')` retourne aussi le hachage là où les lectures directes de catalogue sont bloquées :

```sql
' UNION SELECT name,master.dbo.fn_varbintohexstr(CAST(LOGINPROPERTY(name,'PasswordHash') AS varbinary(256))),NULL FROM sys.sql_logins-- 
```

Lire ces données requiert `CONTROL SERVER` ou `sysadmin` (les colonnes de hachage de mot de passe sont protégées), confirmez donc d'abord le rôle. Les logins authentifiés par Windows n'ont pas de hachage stocké ici ; cette voie ne récupère que les identifiants d'authentification SQL.

## Outils

- hashcat (modes de hachage SQL Server)

## Références

- Microsoft SQL Server Documentation : sys.sql_logins, LOGINPROPERTY
- OWASP Testing Guide : Testing for SQL Injection
