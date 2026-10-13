---
order: 9
title: "Attaques par troncature SQL dans MySQL"
description: "Abuser de la troncature de chaînes de MySQL et du retrait des espaces en fin de valeur pour entrer en collision avec un compte existant comme admin, quand le mode SQL strict est désactivé."
keywords:
  - attaque par troncature SQL
  - troncature par longueur de colonne
  - espace de fin
  - collision de nom d'utilisateur
  - prise de contrôle de compte
---

# Troncature

Une attaque par troncature abuse de la façon dont MySQL a stocké une chaîne surdimensionnée dans une colonne de longueur fixe. Lorsque le mode SQL strict est désactivé, insérer une valeur plus longue que la colonne la tronque silencieusement à la largeur de la colonne au lieu de lever une erreur, et les comparaisons `CHAR`/`VARCHAR` ignorent les espaces de fin. Ensemble, ces comportements permettent à une valeur forgée d'entrer en collision avec une ligne existante après stockage.

La cible classique est l'inscription sur une colonne `username VARCHAR(n)`. Inscrire un nom composé du nom de la victime, de suffisamment d'espaces pour dépasser la largeur de la colonne, et d'un caractère final stocke une valeur tronquée égale au nom de la victime :

```
username = "admin                 x"   (padded beyond VARCHAR length)
```

Si l'application vérifie d'abord « est-ce que `admin` existe déjà ? » par rapport à la chaîne complète soumise (qui diffère) puis l'insère, MySQL la tronque et la raccourcit à `admin`, produisant une seconde ligne qui se connecte en tant qu'administrateur avec un mot de passe défini par l'attaquant.

Deux conditions préalables doivent toutes deux être réunies. Premièrement, un `sql_mode` non strict : depuis MySQL 5.7 le mode strict est activé par défaut et transforme l'insertion surdimensionnée en erreur plutôt qu'en troncature, de sorte que l'attaque s'applique aux serveurs plus anciens ou reconfigurés en mode permissif (confirmez avec `SELECT @@sql_mode`). Deuxièmement, une collation `PAD SPACE`, afin que les espaces de fin stockés soient ignorés à la comparaison et que la valeur rembourrée corresponde au nom simple à la connexion. C'était la norme dans les versions plus anciennes, mais les collations par défaut de MySQL 8.0 (`utf8mb4_0900_ai_ci` et le reste de l'ensemble `utf8mb4_0900_*`) sont `NO PAD`, ce qui rend les espaces de fin significatifs et casse la collision, de sorte qu'une colonne sur une collation `PAD SPACE` telle que `utf8mb4_general_ci` est également requise.

## Outils

- Tests manuels avec Burp Repeater et le client mysql.

## Références

- MySQL Reference Manual : `sql_mode`, string column storage and comparison
- OWASP Testing Guide : Testing for SQL Injection
