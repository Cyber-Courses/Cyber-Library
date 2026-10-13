---
order: 2
title: "Server-Sent Events"
description: "Injection de saut de ligne dans un text/event-stream où les lignes data, event et id construites par le serveur proviennent d'un état non fiable, contrefaisant des événements et empoisonnant des flux partagés."
keywords:
  - Server-Sent Events
  - injection SSE
  - event-stream
  - injection de saut de ligne
  - empoisonnement de flux
---

# Server-Sent Events

Les Server-Sent Events poussent un flux unidirectionnel de texte du serveur vers le client sur une seule réponse à longue durée de vie avec le type de contenu `text/event-stream`. Le format filaire est volontairement simple : chaque événement est un groupe de lignes comme `data:`, `event:` et `id:`, et une ligne vide termine un événement et démarre le suivant. Cette simplicité est la faiblesse. Les frontières entre les événements sont de simples sauts de ligne dans le corps, donc lorsque le serveur construit ces lignes à partir d'un état non fiable sans retirer les sauts de ligne, l'entrée de l'attaquant peut écrire sa propre structure de flux.

## Le format du flux

Un événement bien formé ressemble à ceci :

```
event: message
data: {"user": "alice", "text": "hello"}
id: 42

```

Le parseur `EventSource` du client découpe sur les sauts de ligne et dispatche à la frontière de la ligne vide. Le parseur n'a aucun framing au-delà de ces caractères, donc quiconque contrôle une séquence d'octets incluant `\n` contrôle où les événements commencent et finissent.

## Injection de saut de ligne pour contrefaire des événements

Supposons que le serveur reflète un nom d'utilisateur ou un message dans une ligne `data:` :

```
data: {"from": "USERINPUT", "text": "..."}
```

Si `USERINPUT` peut contenir un saut de ligne, une valeur confectionnée ferme l'événement courant et en injecte un second, entièrement choisi par l'attaquant. En fournissant ceci comme nom d'utilisateur :

```
alice"}

event: adminNotice
data: {"text":"Account verified. Visit https://attacker.example to confirm."}
```

cela produit deux événements sur le flux : l'original tronqué et un `adminNotice` contrefait que le client dispatche comme si le serveur l'avait émis. Les clients qui enregistrent des handlers par type d'événement (`source.addEventListener('adminNotice', ...)`) agiront sur le type injecté.

## Découpage et contrebande d'enregistrements

La même primitive découpe un enregistrement logique en plusieurs, ou fusionne les frontières prévues. Injecter une ligne vide termine prématurément un événement avant l'intention du serveur, supprimant des champs attendus par le client et désynchronisant un parseur qui suppose une structure fixe. Injecter une ligne `id:` fixe le last-event-id du flux :

```
value

id: 99999
data: injected
```

Comme le client renvoie le dernier id dans l'en-tête `Last-Event-ID` lorsqu'il se reconnecte, un `id:` contrefait peut planter une valeur à laquelle le serveur fait confiance à la reprise, atteignant toute recherche qui consomme cet en-tête lors de la reconnexion.

## Empoisonnement d'un flux partagé ou relayé

L'impact se multiplie lorsque le flux est partagé. Un fil de notifications ou d'activité qui diffuse un `text/event-stream` côté serveur vers de nombreux abonnés, ou un proxy qui relaie un flux amont aux clients connectés, transportera un événement injecté à chaque destinataire. Une seule valeur fournie par l'attaquant contenant des lignes d'événement délimitées par des sauts de ligne devient un événement contrefait livré à toute l'audience de ce channel :

```
ok

event: priceUpdate
data: {"symbol":"ACME","price":0.01}
```

Chaque abonné dont le client agit sur `priceUpdate` reçoit l'enregistrement contrefait. Lorsque le relais stocke aussi les événements pour un replay, les lignes contrefaites persistent et sont re-servies aux clients qui se connectent plus tard.

## Trouver le sink

Identifiez tout champ que le serveur reflète dans le flux : noms d'utilisateur, corps de chat, chaînes de statut, noms de ressources. Soumettez une valeur portant `\n`, `\r\n`, une ligne vide et un second bloc `data:`/`event:`/`id:`, puis observez le flux brut (un simple `curl -N` contre l'endpoint montre les octets non parsés). Un flux où vos lignes injectées apparaissent comme leurs propres événements, plutôt qu'échappées à l'intérieur de la valeur `data:` originale, confirme le sink.

## Outils

- **curl -N** : diffuser la réponse `text/event-stream` pour observer les octets bruts, non parsés.
- **Burp Suite** : intercepter et altérer les requêtes qui alimentent les champs reflétés dans le flux.
- Tests manuels avec Burp Repeater et payloads confectionnés.

## Références

- OWASP Web Security Testing Guide: Testing for HTTP Splitting and Smuggling
- PortSwigger Web Security Academy: HTTP response header injection
