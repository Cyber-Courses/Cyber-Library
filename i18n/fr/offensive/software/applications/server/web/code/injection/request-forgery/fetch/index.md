---
order: 5
title: "Fetch : le client qui effectue la requête SSRF"
description: "Orthogonal à la grammaire de l'URL, le client qui émet la requête sortante décide de ce qu'une URL malveillante peut faire. Une bibliothèque HTTP programmatique et un navigateur headless se comportent très différemment pour la même URL."
keywords:
  - client SSRF
  - SSRF navigateur headless
  - client HTTP programmatique
  - suivi de redirections
  - réutilisation de cookies
---

# Fetch

Les sous-arbres [Authority](../authority/index.md), [Path](../path.md), [Query](../query/index.md) et [Scheme](../scheme/index.md) décrivent *quelle URL* envoyer. Fetch est l'autre axe : *ce qui effectue la requête*. La même URL malveillante produit des résultats différents selon le client, donc le client vaut la peine d'être classé avant de choisir un payload.

- **[Programmatique](programmatic.md)** : un client HTTP de bibliothèque tel que `fetch`, `axios`, `got`, Java `HttpClient`, Python `requests`/`httpx`, ou Go `net/http`. Il envoie la requête sans contexte d'exécution JavaScript et généralement sans session interactive, donc la surface d'attaque est l'URL, la politique de redirection et les en-têtes que le serveur attache.
- **[Navigateur headless](headless-browser.md)** : un moteur de navigateur complet piloté côté serveur (Puppeteer, Playwright, Selenium) pour le rendu, les captures d'écran ou l'export PDF. Il transporte cookies et stockage local, exécute du script et suit les chargements de sous-ressources, de sorte qu'une seule navigation peut réutiliser une session authentifiée, exécuter du HTML attaquant et enchaîner des requêtes.

## Pourquoi le client compte

Un client programmatique qui refuse les redirections et rejette les schemes non-`http` est une cible étroite. Un navigateur headless pointé vers la même URL est bien plus large : il rend le markup attaquant (de sorte que le HTML et le script injectés s'exécutent dans un contexte privilégié), il charge `file://` et des sous-ressources internes, et il peut attacher un vrai cookie de session à la requête. Décider lequel l'application utilise vous indique s'il faut investir dans des payloads de scheme à octets bruts (programmatique) ou dans la réutilisation de session et l'exfiltration par script rendu (headless). Les astuces au niveau hôte telles que le [DNS rebinding](../authority/domain-name/dns-rebinding.md) s'appliquent aux deux, car les deux résolvent des noms.

## Outils

- **SSRFmap** : piloter les payloads une fois le type de client connu.
- **Burp Suite** : identifier le comportement du client (redirections, user agent, exécution de script) par requête.
- **Burp Collaborator** : distinguer les clients en observant quels chargements out-of-band se déclenchent.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
