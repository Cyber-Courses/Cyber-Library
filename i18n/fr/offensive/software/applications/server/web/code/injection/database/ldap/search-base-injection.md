---
order: 3
title: "Injection de base de recherche LDAP"
description: "Contrôler la base de recherche ou la portée qu'une application transmet à une requête LDAP pour qu'elle lise une partition plus large ou différente de l'annuaire que prévu."
keywords:
  - injection de base de recherche
  - base DN
  - portée subtree
  - partition LDAP
  - lecture d'annuaire
---

# Injection de base de recherche

Une recherche LDAP prend trois entrées : la base DN (où commencer), la portée (base, un niveau, ou subtree) et le filtre. Les applications fixent généralement la base et la portée et ne font varier que le filtre, mais lorsque la base ou la portée est construite à partir d'une entrée, un attaquant peut rediriger la recherche pour lire davantage de l'arbre.

Si la base est assemblée sous la forme `ou=$dept,dc=example,dc=com`, injecter des RDN dans `$dept` ne peut que déplacer le point de départ latéralement ou plus en profondeur dans ce suffixe fixe (les DN LDAP n'ont pas de syntaxe de remontée vers le parent, donc vous ne pouvez pas remonter au-dessus de `dc=example,dc=com` depuis un fragment), et les RDN supplémentaires ne font que s'ajouter en dessous :

```
dept input: staff,ou=contractors
base: ou=staff,ou=contractors,dc=example,dc=com
```

Le cas à fort impact est celui où l'application prend l'intégralité de la base DN, ou la portée, auprès du client plutôt qu'un simple fragment. Une base fournie par le client correspondant à la racine `dc=example,dc=com` avec une portée subtree transforme une requête censée lister un seul département en une énumération de chaque entrée sous le domaine. De même, lorsque la portée est paramétrée, passer une recherche `base`/`one-level` à `subtree` élargit ce qu'une seule requête renvoie. La primitive dépend donc de la part de la base et de la portée que l'entrée contrôle : un fragment en suffixe redirige à l'intérieur d'un sous-arbre, tandis qu'une base ou une portée complète élargit à l'annuaire entier.

L'injection de base de recherche ne change pas la logique du filtre (cela, c'est l'injection de filtre) mais change la *région* sur laquelle le filtre s'exécute, de sorte qu'un filtre inoffensif correspond soudain à travers tout l'annuaire. Elle mène à une divulgation d'informations horizontale, lisant des entrées dans des partitions dont l'application était écartée par sa portée. La défense, qu'il ne vaut la peine de mentionner que pour expliquer le bug, est que la base DN et la portée devraient être fixées côté serveur et jamais prises auprès du client ; lorsqu'elles ne le sont pas, c'est la primitive.

## Outils

- **ldapsearch** : émettre des recherches avec une base DN et une portée variées pour confirmer l'élargissement.
- **windapsearch** : énumérer l'arbre accessible depuis une base redirigée.
- **Burp Repeater** : forger des requêtes qui contrôlent le paramètre de base ou de portée.

## Références

- RFC 4511 : LDAP search operation (baseObject, scope)
- OWASP Testing Guide: Testing for LDAP Injection
