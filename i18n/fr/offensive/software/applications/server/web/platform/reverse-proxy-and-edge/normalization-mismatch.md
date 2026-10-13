---
order: 1
title: "Désaccord de normalisation sur le proxy : confusion de chemin qui contourne le contrôle d'accès"
description: "Exploiter une canonicalisation de chemin différente entre un reverse proxy et son origin : slashs et dot-segments encodés, et paramètres de chemin qui passent une règle d'autorisation/refus de l'edge tout en se résolvant en un chemin protégé au niveau de l'origin."
keywords:
  - confusion de chemin
  - désaccord de normalisation
  - contournement de reverse proxy
  - slash encodé
  - contournement de contrôle d'accès
---

# Désaccord de normalisation sur le proxy

Un reverse proxy applique souvent des règles d'accès sur le chemin de la requête (« refuser `/admin` », « exiger une authentification sous `/internal` ») puis transmet à l'origin. Si le proxy et l'origin **canonicalisent le chemin différemment**, un chemin forgé paraît autorisé au proxy mais se résout en la ressource protégée au niveau de l'origin, contournant le contrôle.

## Le mécanisme

Le proxy confronte sa règle à une interprétation du chemin ; l'origin fait correspondre une interprétation différente à un fichier ou une route. Divergences fréquentes :

- **Slashs/points encodés** : le proxy ne décode pas `%2f`/`%2e` mais l'origin le fait, de sorte que `/public/..%2fadmin` passe une règle « refuser /admin » et se résout en `/admin`.
- **Paramètres de chemin** : `/admin;foo=bar` ou `/..;/admin` sont distincts pour le proxy mais retirés par un origin Java/Tomcat (voir [paramètres de chemin de servlet](../tomcat-and-java/path-parameter-and-normalization.md)).
- **Moment de traitement des dot-segments** : un étage réduit `/a/../admin` en `/admin` et l'autre non.
- **Contenu en fin de chemin** : `/admin/..%2f..%2fadmin`, des doubles slashs, ou un `/.` final que seul un des côtés normalise.

```
GET /public/..%2fadmin/ HTTP/1.1          # edge sees /public/..%2fadmin, origin sees /admin
GET /admin..;/ HTTP/1.1
GET /%2561dmin/ HTTP/1.1                   # stacked with double-encoding
```

## Comment le repérer

- Identifier un chemin que l'edge protège (401/403/redirection sur `/admin`, `/internal`, `/actuator`, `/manager`).
- Le rejouer enveloppé dans des formes que l'edge est peu susceptible de normaliser mais que l'origin normalisera : `..%2f`, `;/`, `%2e%2e`, doubles slashs, casse mixte, et combinaisons.
- Un corps de ressource protégée renvoyant un `200` via un chemin forgé confirme le désaccord.

## Exploitation

- Atteindre les consoles d'administration, les actuators/endpoints de gestion de frameworks et les API internes que l'edge croyait avoir cloisonnés (y compris le [Tomcat Manager](../tomcat-and-java/manager-and-host-manager.md)).
- Combiner avec l'[exposition de l'origin](origin-exposure.md) : si le contrôle n'existe qu'au niveau de l'edge, une désynchronisation vers la route protégée a le même effet que de parler directement à l'origin.
- Combiner avec le [double décodage](../iis/double-decode-and-unicode-traversal.md) lorsque l'edge décode une fois.

## Outils

- Burp (Repeater/Intruder) avec des payloads de confusion de chemin ; **curl --path-as-is**.

## Références

- PortSwigger Web Security Academy: Access control bypass
- Orange Tsai: reverse-proxy path confusion research
