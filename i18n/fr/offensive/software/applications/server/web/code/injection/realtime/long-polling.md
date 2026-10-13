---
order: 3
title: "Long polling"
description: "Endpoints maintenus ouverts et de polling où les notifications JSON, les paramètres de requête du poll et les jetons de session dans l'URL du poll atteignent les requêtes et l'autorisation côté serveur."
keywords:
  - long polling
  - comet
  - endpoint de polling
  - jeton dans l'URL
  - injection
---

# Long polling

Le long polling émule un canal push par-dessus du HTTP ordinaire : le client émet une requête que le serveur maintient ouverte jusqu'à ce que des données soient disponibles ou qu'un timeout se déclenche, puis le client ré-émet immédiatement une requête. Chaque cycle est une requête HTTP normale, ce qui signifie que les endpoints de long polling héritent de toute la surface d'attaque REST, plus deux problèmes propres au modèle : les valeurs qui pilotent le poll (cursor, channel, token) voyagent à chaque cycle et se trouvent souvent dans l'URL, et la boucle maintien-ouvert / ré-émission expose un comportement d'ordonnancement et de timing qu'un attaquant peut orienter.

## Injection via les paramètres de poll

Une requête de poll transporte un état indiquant au serveur ce que le client veut ensuite, typiquement un nom de channel, un cursor et un since-timestamp :

```http
GET /poll?channel=user:42&since=1696000000&cursor=a1b2 HTTP/1.1
Host: target.example
Cookie: session=...
```

Chacun de ces paramètres est un sink candidat. Si `channel` est concaténé dans une requête qui sélectionne les messages en attente, il transporte du SQL :

```
/poll?channel=user:42' UNION SELECT secret FROM tokens-- -
```

Si `since` ou `cursor` alimente une recherche dans un document store, un objet opérateur soumis en JSON dans un poll basé sur POST contourne le filtre :

```json
{"channel": "user:42", "cursor": {"$gt": ""}}
```

Comme les endpoints de long polling sont sollicités en continu et sont souvent considérés comme de la plomberie de faible valeur, leurs paramètres sont fréquemment moins scrutés que les routes REST principales tout en atteignant les mêmes backends.

## Le channel comme sink de contrôle d'accès

Le paramètre `channel` est aussi une frontière d'autorisation. Si le serveur renvoie n'importe quel channel nommé par le poll sans vérifier que la session a le droit de le lire, échanger le channel permet de lire les notifications en attente d'autres utilisateurs via le propre poll authentifié de l'attaquant :

```
/poll?channel=user:43
/poll?channel=admin:events
/poll?channel=tenant:acme:billing
```

C'est la forme « endpoint de poll » du broken object-level authorization : l'identité est vérifiée, le channel nommé ne l'est pas.

## Fuite de jeton dans l'URL

Les clients de long poll transportent souvent la session ou un jeton par channel dans l'URL du poll elle-même, parce que le modèle est antérieur à des mécanismes d'authentification de transport plus propres et les contourne :

```
/poll?channel=user:42&access_token=eyJhbGciOi...
```

Un jeton dans la query string fuit à travers chaque couche qui enregistre les URL de requête : les journaux d'accès des proxies et des serveurs, et les outils d'APM ou d'analytics qui capturent les URL complètes. Ce sont de vrais points de collecte, et un journal d'accès partagé ou ingéré livre le jeton à quiconque peut lire les journaux. Les fuites côté navigateur (historique et `Referer` vers des ressources tierces) ne s'appliquent que lorsque le jeton se trouve dans l'**URL de page de premier niveau** : pour un poll via XHR ou `fetch`, l'URL de requête n'est jamais une navigation, donc elle n'est pas ajoutée à l'historique, et une ressource tierce ultérieure reçoit l'URL de la page conteneur comme `Referer`, pas l'URL du poll.

## Ordonnancement et timing

La boucle maintien-ouvert / ré-émission crée une surface de course. Comme le client déclenche un nouveau poll à l'instant où le précédent revient, deux polls peuvent se chevaucher durant l'intervalle de ré-émission, et un attaquant qui contrôle à la fois le poll et une requête concurrente modifiant l'état peut les entrelacer pour lire une valeur en cours de mise à jour ou recevoir une notification censée être filtrée derrière une vérification pas encore validée. L'ordre de livraison est également observable : en mesurant quel poll renvoie en premier un enregistrement donné, un attaquant peut inférer le séquençage côté serveur et le timing d'événements dans les sessions d'autres utilisateurs qui partagent une file backend.

## Trouver les sinks

Capturez un cycle de poll, puis énumérez chaque paramètre contre les mêmes payloads utilisés pour les routes REST, et séparément échangez le channel et le cursor vers des valeurs voisines et privilégiées. Vérifiez si la session ou un jeton voyage dans l'URL plutôt que dans un cookie ou un en-tête ; si c'est le cas il fuit vers les journaux serveur et proxy, et en plus via l'historique du navigateur ou le `Referer` uniquement lorsque ce même jeton apparaît aussi dans l'URL de page de premier niveau.

## Outils

- **Burp Suite** : intercepter un cycle de poll, puis fuzzer et rejouer les paramètres avec Burp Repeater.
- **curl** : émettre des requêtes de poll directement pour échanger les valeurs de channel, cursor et token.

## Références

- PortSwigger Web Security Academy: Access control vulnerabilities and privilege escalation
- OWASP Web Security Testing Guide: Testing for Insecure Direct Object References
