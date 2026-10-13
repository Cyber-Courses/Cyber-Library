---
order: 4
title: "Injection LDAP : abus de filtre, de DN et de base de recherche"
description: "Exploitation de l'injection LDAP lorsque le code applicatif construit des filtres d'annuaire, des noms distinctifs ou des bases de recherche à partir d'une entrée non fiable, y compris le contournement d'authentification et l'extraction en aveugle."
keywords:
  - injection LDAP
  - injection de filtre LDAP
  - injection de DN
  - injection de base de recherche
  - requête d'annuaire
---

# LDAP

L'injection LDAP survient lorsqu'une application construit une opération d'annuaire à partir d'une entrée non fiable sans l'échapper, l'équivalent pour l'annuaire de l'injection SQL. Elle apparaît partout où un filtre, un nom distinctif (DN) ou une base de recherche est assemblé par concaténation de chaînes.

La syntaxe centrale est le filtre de recherche, écrit en notation préfixée : `(attr=value)`, combiné avec `&` (AND), `|` (OR) et `!` (NOT), comme dans `(&(uid=jane)(objectClass=person))`. Le caractère générique `*` correspond à n'importe quelle valeur. La RFC 4515 exige l'échappement de cinq caractères à l'intérieur d'une valeur de filtre (`*` `(` `)` `\` et NUL, sous la forme `\2a \28 \29 \5c \00`) ; lorsque l'application ne les échappe pas, les parenthèses et caractères génériques injectés changent la logique du filtre. Les DN ont leurs propres règles d'échappement (RFC 4514) pour des caractères comme `,` `+` `=` et `"`.

Contrairement à SQL, LDAP n'a ni `UNION` ni commentaires, et les erreurs sont généralement laconiques, donc l'extraction s'appuie sur les caractères génériques et les différences de réponse booléennes plutôt que sur une riche sortie en bande. L'impact va du contournement d'authentification et des failles d'autorisation à la divulgation d'attributs d'annuaire.

## Techniques

- **[Injection de filtre](filter-injection.md)** : injecter des parenthèses, des opérateurs et des caractères génériques dans un filtre de recherche.
- **[Contournement d'authentification](authentication-bypass.md)** : détourner une connexion adossée à LDAP.
- **[Aveugle](blind.md)** : déduire les valeurs d'attributs à partir des réponses par caractère générique et booléennes.
- **[Injection de DN](dn-injection.md)** : altérer le nom distinctif utilisé pour le bind ou la modification.
- **[Injection de base de recherche](search-base-injection.md)** : élargir ou déplacer la base de recherche pour lire davantage de l'arbre.

## Outils

- **ldapsearch** : test manuel de filtre, de DN et de base contre l'annuaire.
- **windapsearch** : énumérer les utilisateurs, groupes et attributs via LDAP.
- **python-ldap** : scripter les opérations de bind et de recherche pour des payloads personnalisés.
- **Burp Suite** : intercepter et manipuler les requêtes adossées à l'annuaire.

## Références

- RFC 4515 (LDAP search filters) et RFC 4514 (distinguished names)
- OWASP Testing Guide: Testing for LDAP Injection
