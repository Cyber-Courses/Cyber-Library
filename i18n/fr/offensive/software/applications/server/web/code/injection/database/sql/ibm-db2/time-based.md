---
order: 9
title: "Injection SQL aveugle temporelle dans IBM Db2"
description: "Inférer les données IBM Db2 à partir de délais conditionnels construits avec une requête lourde, puisque Db2 n'a pas de fonction SLEEP ou WAITFOR."
keywords:
  - blind temporel
  - requête lourde
  - pas de SLEEP
  - cross join
  - timing Db2
---

# Temporelle

Db2 n'a pas de `SLEEP` ni de `WAITFOR` (`WAITFOR` est du T-SQL, une confusion courante), donc un délai est produit en forçant la base de données à effectuer un travail coûteux uniquement lorsqu'une condition est vraie. Un grand cross join sur une vue de catalogue est le dispositif habituel.

Joindre une grande vue de catalogue à elle-même plusieurs fois produit un énorme résultat intermédiaire, et le compter coûte un CPU mesurable. Conditionnez-le avec `CASE` pour que le coût ne soit payé que sur un test vrai :

```sql
' AND (SELECT CASE WHEN (ASCII(SUBSTR((SELECT CURRENT USER FROM SYSIBM.SYSDUMMY1),1,1))>77) THEN (SELECT COUNT(*) FROM SYSCAT.COLUMNS a, SYSCAT.COLUMNS b, SYSCAT.COLUMNS c) ELSE 1 END FROM SYSIBM.SYSDUMMY1)>0-- 
```

Quand le test est vrai, Db2 évalue le triple cross join (souvent plusieurs secondes sur un catalogue normal) ; quand il est faux, le `ELSE` retourne immédiatement. Ajustez le nombre de copies jointes, ou utilisez une vue plus grande telle que `SYSIBM.SYSCOLUMNS`, pour qu'un test vrai ajoute un délai clair et reproductible sans surcharger le serveur.

Recherchez chaque caractère par recherche binaire sur le délai exactement comme dans l'extraction booléenne. Parce que le délai est lié au CPU plutôt qu'un sleep fixe, il est plus bruité que `pg_sleep` ou `WAITFOR`, donc gardez le travail assez volumineux pour être lu clairement et répétez un hit positif avant de lui faire confiance. Vu le canal d'erreur faible de Db2, l'inférence booléenne est généralement préférée, avec ce timing par requête lourde comme solution de repli lorsqu'aucune différence booléenne n'est visible.

## Outils

- **sqlmap** : extraction temporelle automatisée avec payloads de cross join lourds (`--technique=T`).
- **ghauri** : inférence temporelle rapide avec forte évasion de WAF.
- **Burp Repeater** : mesurez le délai conditionnel à la main pour ajuster le nombre de jointures.

## Références

- IBM Db2 SQL Reference : subselect, joins, CASE, catalog views
- PortSwigger Web Security Academy : Blind SQL injection
