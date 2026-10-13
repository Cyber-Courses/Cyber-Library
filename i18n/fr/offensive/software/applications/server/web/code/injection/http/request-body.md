---
order: 2
title: "Corps de requête HTTP et confusion de content-type"
description: "Abuser de la façon dont les serveurs analysent le corps de requête HTTP : confusion de content-type, précédence corps contre query, et gestion multipart pour contourner la validation et atteindre des parseurs inattendus."
keywords:
  - confusion de content-type
  - analyse du corps de requête
  - multipart
  - mass assignment
  - précédence de paramètre
---

# Corps de requête

La façon dont un serveur interprète le corps de requête dépend du `Content-Type` qu'on lui annonce, et les décalages entre ce que le client envoie, ce qu'un filtre inspecte et ce que le framework analyse créent des contournements.

La confusion de content-type envoie le corps dans un format que la cible n'attend pas mais analyse tout de même. Un framework qui accepte à la fois le form-encoded et le JSON peut appliquer une liaison ou une validation différente à chacun, donc changer de `Content-Type` peut atteindre un parseur plus permissif ou permettre le mass assignment de champs que le chemin du formulaire rejetterait :

```
POST /api/profile HTTP/1.1
Content-Type: application/json

{"name":"x","is_admin":true}
```

là où le même endpoint, lu comme un formulaire, aurait ignoré `is_admin`. Un WAF qui n'inspecte que les corps de formulaire peut aussi manquer un payload livré en JSON ou XML.

La précédence corps contre query est un bug connexe : de nombreux frameworks fusionnent les paramètres de query et de corps, et lorsque les deux définissent le même nom, une portée l'emporte. Fournir la valeur autorisée dans la portée inspectée et la valeur de l'attaquant dans la portée utilisée écrase une décision après validation.

Les corps multipart ajoutent leur propre surface : analyse de frontière incohérente, noms de parties dupliqués, et les champs `filename`/`Content-Type` à l'intérieur d'une partie peuvent faire passer des valeurs devant la validation ou orienter la gestion des fichiers. Les déclarations de charset (`charset=`) peuvent changer la façon dont les octets se décodent, ressuscitant à l'occasion des caractères filtrés.

L'approche consiste à renvoyer une requête avec le corps réencodé sous un content type différent, avec des paramètres dupliqués à travers les portées, et avec des parties multipart manipulées, en guettant une différence de validation ou de liaison. La portée dépend des parseurs que le framework active, donc confirmez ce que la cible accepte.

## Outils

- **Burp Repeater** : réencoder les corps sous différents content types et manipuler les parties multipart.
- **Turbo Intruder** : scripter des variations à haut volume du content type et de la portée des paramètres.
- **curl** : renvoyer des requêtes avec des en-têtes `Content-Type` alternatifs et des corps forgés.

## Références

- OWASP Testing Guide: Testing for Content Type and parameter handling
- RFC 9110: HTTP content and media types
