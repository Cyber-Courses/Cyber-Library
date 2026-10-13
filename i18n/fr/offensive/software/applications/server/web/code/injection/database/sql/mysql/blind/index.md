---
order: 1
title: "Injection SQL aveugle basée sur booléen dans MySQL"
description: "Déduire les données MySQL un caractère à la fois à partir des différences vrai/faux dans la réponse quand aucune ligne ni erreur n'est reflétée."
keywords:
  - blind basé sur booléen
  - injection SQL aveugle
  - inférence MySQL
  - SUBSTRING ASCII
  - extraction par recherche dichotomique
---

# Blind booléen

Quand l'application ne reflète ni lignes ni texte d'erreur, l'injection fuite tout de même des données à travers son comportement. Si une condition vraie et une condition fausse produisent des réponses observablement différentes (une page, une longueur, un statut ou un message différent), chaque comparaison injectée répond à une question oui/non, et assez de questions reconstruisent n'importe quelle valeur.

Le bloc de base est une comparaison dont la véracité dépend d'un seul caractère de la cible. `SUBSTRING` isole le caractère et `ASCII` le transforme en nombre à comparer :

```sql
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>77-- 
```

Une recherche dichotomique trouve chaque caractère en environ sept requêtes : divisez la plage par deux à chaque réponse jusqu'à ce que le point de code exact soit connu, puis passez à la position deux, trois, et ainsi de suite. Les noms de la base courante, de l'utilisateur et du schéma se lisent de la même façon, en remplaçant la sous-requête interne.

L'extraction aveugle est lente mais fiable, et c'est le repli chaque fois que des canaux plus riches (union, erreur, out-of-band) sont fermés. Les pages ci-dessous couvrent l'oracle de caractères standard et comment le maintenir opérationnel quand des caractères courants sont filtrés.

## Pages

- **[Extraction booléenne](boolean-based.md)** : l'oracle de caractères `SUBSTRING`/`ASCII` avec recherche dichotomique.
- **[Extraction sans virgule](comma-free-extraction.md)** : `LIKE` et `SUBSTRING ... FROM ... FOR` quand les virgules sont bloquées.

## Outils

- **sqlmap** : extraction automatisée en blind booléen.
- **ghauri** : alternative rapide avec forte évasion des WAF.

## Références

- MySQL Reference Manual: SUBSTRING, ASCII, string comparison
- PortSwigger Web Security Academy: Blind SQL injection
