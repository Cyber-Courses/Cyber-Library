---
order: 2
title: "Injection de DN LDAP"
description: "Altérer le nom distinctif qu'une application construit pour une opération de bind ou de modification en injectant des séparateurs de RDN et des métacaractères de DN non échappés."
keywords:
  - injection de DN
  - nom distinctif
  - séparateur de RDN
  - RFC 4514
  - bind DN
---

# Injection de DN

Un nom distinctif identifie de manière unique une entrée d'annuaire, par exemple `uid=jane,ou=people,dc=example,dc=com`. Lorsqu'une application assemble un DN à partir d'une entrée (pour se lier en tant qu'utilisateur, ou pour localiser une entrée à lire ou à modifier), une entrée non échappée peut changer l'entrée vers laquelle pointe le DN.

Les DN utilisent un jeu de métacaractères différent de celui des filtres (RFC 4514) : `,` `+` `=` `"` `\` `<` `>` `;`, un `#` initial, ainsi que les espaces en début ou en fin doivent être échappés. La virgule et `+` sont les plus dangereux, car ils séparent les noms distinctifs relatifs (RDN) et combinent les RDN multivalués.

Contre un gabarit `uid=$input,ou=people,dc=example,dc=com`, injecter une virgule reparentre ou réoriente l'entrée :

```
input: jane,ou=admins
DN: uid=jane,ou=admins,ou=people,dc=example,dc=com
```

Cela déplace le DN vers un sous-arbre différent, de sorte qu'un bind ou une recherche aboutit à une entrée que le développeur n'avait pas prévue. Un RDN multivalué via `+` attache une autre assertion attribut-valeur au nom (par exemple `uid=jane+cn=admin`). À noter qu'un `=` supplémentaire dans la valeur ne redéfinit pas le type d'attribut du RDN : la RFC 4514 scinde le type au premier `=` seulement, donc tout `=` ultérieur reste partie intégrante de la valeur `uid`. La virgule (reparentage) et `+` (RDN multivalué) sont les primitives effectives.

L'injection de DN est plus étroite que l'injection de filtre, car un DN doit toujours se résoudre en une entrée réelle, donc elle sert à pivoter vers une entrée voisine ou à privilèges plus élevés (par exemple déplacer un bind DN dans une OU administrative) plutôt qu'à faire correspondre largement. Lorsque l'entrée alimente le bind DN d'une étape d'authentification, la contrôler peut sélectionner le compte sous lequel l'application se lie.

## Outils

- **ldapsearch** : résoudre des DN candidats pour confirmer que le reparentage aboutit à une entrée réelle.
- **windapsearch** : énumérer les OU et les entrées pour choisir un DN cible à privilèges plus élevés.
- **Burp Repeater** : forger des requêtes qui alimentent le fragment de DN injecté.

## Références

- RFC 4514 : LDAP string representation of distinguished names
- OWASP Testing Guide: Testing for LDAP Injection
