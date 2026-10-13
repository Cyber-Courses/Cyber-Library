---
order: 3
title: "Injection de log JSON : injecter des clés dans des logs structurés construits par concaténation"
description: "Lorsqu'une ligne de log JSON est assemblée par concaténation de chaînes, injecter un guillemet et une virgule pour ajouter ou écraser des clés comme level, user et trace, casser le schéma de l'agrégateur, ou faire passer en fraude des objets imbriqués dans ELK ou Loki."
keywords:
  - injection de log
  - injection de log JSON
  - logging structuré
  - injection ELK
  - injection Loki
  - empoisonnement de schéma
---

# Injection de log JSON

Le logging structuré émet un objet JSON par ligne, consommé par des agrégateurs tels qu'Elasticsearch, Loki, ou un service de logs cloud. Lorsque le JSON est produit par un vrai sérialiseur, les guillemets intégrés sont échappés et l'objet est sûr. Lorsqu'il est au contraire assemblé par concaténation de chaînes, une valeur contrôlée par l'attaquant qui contient un `"` peut fermer sa chaîne, et un `,` suivant peut démarrer une nouvelle clé, donnant à l'attaquant le contrôle de la structure de l'objet plutôt que d'un seul champ.

## La primitive

Une ligne de log JSON construite à la main ressemble à :

```
out.write('{"ts":"' + ts + '","level":"INFO","msg":"login for ' + user + '"}')
```

Avec `user` réglé sur une valeur bénigne, la ligne est valide. Réglez `user` sur une valeur qui ferme la chaîne `msg` et ajoute des clés :

```
bob","level":"ERROR","evil":"
```

La ligne émise devient :

```
{"ts":"2026-10-01T09:14:22Z","level":"INFO","msg":"login for bob","level":"ERROR","evil":""}
```

L'objet a maintenant deux clés `level` et une clé `evil` injectée. Un payload injecté compact tel que `","level":"INFO","x":"` suit la même forme : fermer la valeur courante, ajouter une clé, et rouvrir une chaîne pour que le `"}` de fin se ferme encore proprement.

## Écraser des clés par analyse last-wins

Le JSON autorise les clés dupliquées, et la plupart des parseurs conservent la dernière occurrence. Comme les clés injectées apparaissent après les légitimes, elles l'emportent. Cela laisse un attaquant écraser des champs que l'application a posés :

- Usurper `level` pour cacher une activité sous le filtre de sévérité d'un tableau de bord, ou lever un leurre :

```
bob","level":"DEBUG","msg":"
```

- Usurper l'identité pour que l'événement soit attribué ailleurs :

```
bob","user":"administrator","role":"admin","msg":"
```

- Écraser des champs de corrélation tels que `trace`, `request_id`, ou `span_id` pour détacher l'enregistrement d'une vraie piste de requête ou pour entrer en collision avec la trace d'un autre utilisateur :

```
bob","trace":"00000000000000000000000000000000","msg":"
```

## Ajouter des champs que le schéma n'a jamais attendus

Injecter une toute nouvelle clé pollue l'index. Dans Elasticsearch, un nouveau champ est mappé à sa première apparition ; une valeur dont le type entre en conflit avec le mapping existant déclenche une exception de mapping et le document entier est rejeté, de sorte que le vrai événement est abandonné de l'index :

```
bob","status":{"nested":true},"msg":"
```

Ici `status`, s'il était précédemment mappé en keyword, arrive maintenant en tant qu'objet et l'agrégateur refuse la ligne. Répéter des nouvelles clés distinctes peut aussi gonfler le mapping vers sa limite de champs.

## Faire passer en fraude des objets et des tableaux imbriqués

Parce que l'attaquant contrôle du JSON brut, pas seulement un scalaire, une structure imbriquée peut être injectée pour façonner la façon dont l'enregistrement s'affiche dans un visualiseur ou pour porter un payload dans un champ qu'une alerte lit littéralement :

```
bob","tags":["forged","high-priority"],"geo":{"country":"CH","ip":"8.8.8.8"},"msg":"
```

Dans Loki, où chaque ligne est du texte étiqueté par stream, les clés injectées qu'une étape de pipeline en aval analyse avec un parseur `json` deviennent de nouveaux labels ou champs extraits, laissant un attaquant fabriquer des labels que le stream n'a jamais légitimement portés.

## Casser la ligne entièrement

Lorsque le but est d'abandonner un enregistrement plutôt que d'en forger un, une accolade non équilibrée ou un saut de ligne injecté à l'intérieur de la chaîne rend la ligne invalide en JSON. Les ingesteurs stricts écartent la ligne malformée, retirant l'activité qu'elle aurait enregistrée :

```
bob"}%0a{"ts":"forged
```

Cela scinde un objet en un fragment cassé et un second objet partiel, et c'est l'analogue JSON du newline forging contre les logs en texte brut.

## Notes d'encodage

- Via une entrée de formulaire ou de query HTTP, envoyez le guillemet et la virgule littéralement ; là où le sink est lui-même du JSON, fournissez-les sous forme `\"` et `,` pour qu'ils se décodent en structure brute au niveau du logger.
- Un vrai sérialiseur vient à bout de tout ce qui précède, donc cette technique s'applique spécifiquement au logging JSON par concaténation de chaînes ou par format-string.

## Outils

- **Burp Repeater** : injecter des payloads guillemet-et-virgule dans des champs loggés en JSON.
- **curl** : envoyer des valeurs forgées qui ferment des chaînes et ajoutent des clés.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [PayloadsAllTheThings: CRLF and Log Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
