---
order: 15
title: "Injection SQL aveugle temporelle dans Oracle"
description: "Déduire des données Oracle à partir de délais de réponse conditionnels avec la fonction inline DBMS_PIPE.RECEIVE_MESSAGE, puisque DBMS_LOCK.SLEEP est une procédure utilisable uniquement en PL/SQL."
keywords:
  - time based blind
  - DBMS_PIPE.RECEIVE_MESSAGE
  - DBMS_LOCK.SLEEP
  - délai conditionnel
  - timing Oracle
---

# Temporelle

Quand le vrai et le faux se ressemblent, c'est le timing qui porte le signal. Le détail Oracle essentiel est de savoir quelle primitive de délai peut être appelée à l'intérieur d'un `SELECT`.

`DBMS_LOCK.SLEEP` est une procédure, elle ne peut donc pas être utilisée à l'intérieur d'une expression SQL (elle s'exécute uniquement dans un bloc PL/SQL, que les pilotes standards ne vous donnent pas via une simple injection) et elle nécessite aussi un octroi d'exécution explicite. La fonction qui fonctionne en inline est `DBMS_PIPE.RECEIVE_MESSAGE(pipe, timeout)`, qui bloque pendant `timeout` secondes en attente d'un pipe (inexistant) et renvoie un nombre, de sorte qu'elle peut se placer dans un `CASE` :

```sql
' AND 1=(CASE WHEN (ASCII(SUBSTR((SELECT user FROM dual),1,1))>77) THEN DBMS_PIPE.RECEIVE_MESSAGE('a',5) ELSE 1 END)-- 
```

Quand le test est vrai, `RECEIVE_MESSAGE` attend cinq secondes avant de renvoyer ; quand il est faux, le `ELSE` renvoie immédiatement. Effectuez une recherche binaire sur chaque caractère à partir du délai, exactement comme dans l'extraction booléenne. L'exécution de `DBMS_PIPE` est couramment accordée à `PUBLIC`, ce qui en fait la primitive de timing Oracle par défaut.

Là où `DBMS_PIPE` n'est pas exécutable, une requête lourde donne un délai grossier : une grande jointure cartésienne (par exemple `(SELECT COUNT(*) FROM all_objects a, all_objects b, all_objects c)`) consomme un temps mesurable, conditionné par un `CASE`, bien que ce soit bien moins précis que `RECEIVE_MESSAGE`. Maintenez des délais de plusieurs secondes et répétez un résultat positif avant de lui faire confiance.

## Outils

- **sqlmap** : extraction temporelle automatisée avec les charges Oracle `DBMS_PIPE.RECEIVE_MESSAGE` (`--technique=T`).
- **ghauri** : inférence temporelle rapide avec une forte évasion de WAF.
- **Burp Repeater** : mesurer le délai conditionnel à la main pour confirmer la primitive.

## Références

- Oracle Database PL/SQL Packages and Types Reference: DBMS_PIPE, DBMS_LOCK
- PortSwigger Web Security Academy: Blind SQL injection
