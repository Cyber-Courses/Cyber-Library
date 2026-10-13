---
order: 2
title: "Injection XPath"
description: "Une expression XPath construite par concaténation de chaînes sur un document XML permet à un attaquant de réécrire les prédicats et les chemins de localisation pour contourner l'authentification, sélectionner des nœuds non prévus et extraire l'ensemble du document."
keywords:
  - injection XPath
  - injection de requête XML
  - injection de prédicat
  - contournement d'authentification XPath
  - XPath aveugle
---

# XPath

XPath est le langage de requête pour naviguer dans un document XML, et il est injectable pour la même raison que SQL : lorsque le code de l'application concatène l'entrée utilisateur dans une expression au lieu de la lier, l'entrée est parsée comme de la syntaxe de requête. Un attaquant qui atteint un XPath construit par concaténation de chaînes peut s'échapper d'un littéral de chaîne, réécrire le prédicat et rediriger le chemin de localisation vers des nœuds que la requête n'était jamais censée renvoyer.

Une forme vulnérable courante est l'authentification contre un magasin d'utilisateurs XML :

```python
expr = "//user[name='" + username + "' and pass='" + password + "']"
root.xpath(expr)
```

Avec `username` défini à `' or '1'='1`, le prédicat est toujours vrai et la requête renvoie un nœud utilisateur quel que soit le mot de passe. XPath n'a aucun contrôle d'accès et aucune notion de tables séparées : l'ensemble du document est un seul arbre adressable, donc une seule expression injectée peut cheminer du nœud d'authentification vers tous les autres nœuds, étiquettes et attributs du fichier.

L'injection XPath est à plusieurs égards plus permissive que l'injection SQL. Il n'y a pas de concept de connexion privilégiée versus non privilégiée, les commentaires sont rarement nécessaires parce que les expressions sont courtes, et l'opérateur d'union `|` joint des chemins de localisation arbitraires sans schéma à satisfaire. Là où les résultats ne sont pas reflétés, l'inférence booléenne et basée sur les erreurs reconstruit le document caractère par caractère.

Ce sous-arbre couvre l'injection de prédicat et le contournement d'authentification, l'injection de chemin par union à travers l'arbre et l'extraction basée sur les erreurs.

## Pages

- **[Basé sur les erreurs](error-based-xpath.md)** : forcer un échec de cast ou de type dans un processeur XPath 2.0 intègre le texte d'un nœud dans le message d'erreur, transformant un moteur verbeux en un canal d'extraction direct et non...
- **[Injection de prédicat](predicate-injection.md)** : s'échapper d'un littéral de chaîne à l'intérieur d'un prédicat XPath réécrit la condition du filtre pour le contournement d'authentification et l'extraction de nœud à l'aide de logique booléenne et...
- **[Injection de chemin par union](union-path-injection.md)** : l'opérateur d'union | ajoute des chemins de localisation arbitraires à un XPath construit par concaténation de chaînes, sélectionnant des nœuds hors du sous-arbre prévu et dumpant l'ensemble du document.

## Outils

- **xcat** : automatisation de la récupération de données XPath booléenne et basée sur les erreurs.
- **Burp Suite** : fuzzing des points d'injection XPath avec Intruder.
- Tests manuels avec des charges utiles booléennes et d'union dans Burp Repeater.

## Références

- PortSwigger Web Security Academy : XPath injection
- OWASP WSTG : Testing for XPath Injection
