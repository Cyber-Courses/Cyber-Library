---
order: 1
title: "Injection de filtre LDAP"
description: "Injecter des parenthèses, des opérateurs booléens et des caractères génériques dans un filtre de recherche LDAP non échappé pour changer sa logique et lire des entrées non prévues."
keywords:
  - injection de filtre LDAP
  - caractère générique
  - opérateur booléen
  - RFC 4515
  - échappement de filtre
---

# Injection de filtre

Lorsqu'une entrée utilisateur est concaténée dans un filtre de recherche LDAP sans échappement, l'attaquant contrôle la structure du filtre, pas seulement une valeur. Les caractères injectables sont les cinq métacaractères de la RFC 4515, avant tout les parenthèses et le caractère générique `*`.

Considérons un filtre de recherche de personnes `(&(objectClass=person)(cn=$input))`. Un caractère générique renvoie tout :

```
input: *
filter: (&(objectClass=person)(cn=*))
```

Injecter des parenthèses et un opérateur réécrit la logique. Fermer la clause courante et en ajouter une nouvelle permet à l'attaquant d'ajouter des conditions que le développeur n'a jamais écrites :

```
input: *)(uid=admin)
filter: (&(objectClass=person)(cn=*)(uid=admin))
```

Les opérateurs booléens élargissent une correspondance. Fournir une entrée qui ouvre un groupe `|` (OR) rend le filtre vrai pour bien plus d'entrées, et des métadonnées peuvent être divulguées en testant la présence d'attributs :

```
input: *)(|(objectClass=*
filter: (&(objectClass=person)(cn=*)(|(objectClass=*))
```

La manière exacte dont un filtre supplémentaire injecté après le `)` externe est traité dépend de la bibliothèque cliente et du serveur : certains n'utilisent que le premier filtre complet, qui correspond quand même largement, tandis que d'autres rejettent une chaîne mal formée. La primitive fiable est le caractère générique plus des clauses injectées à l'intérieur du groupe existant. Parce que le filtre est structuré en notation préfixée, l'attaquant travaille en fermant des groupes avec `)` et en en ouvrant de nouveaux avec `(`, à l'image de la façon dont l'injection SQL ferme les guillemets, et la même entrée non échappée alimente les techniques de contournement d'authentification et en aveugle.

## Outils

- **ldapsearch** : tester les chaînes de filtre injectées directement contre l'annuaire.
- **windapsearch** : énumérer les objets et attributs d'annuaire dignes d'être ciblés.
- **Burp Repeater** : forger des requêtes avec payloads d'opérateurs et comparer les réponses.
- **python-ldap** : scripter les payloads de filtre de façon programmatique.

## Références

- RFC 4515 : LDAP search filter string representation
- OWASP Testing Guide: Testing for LDAP Injection
