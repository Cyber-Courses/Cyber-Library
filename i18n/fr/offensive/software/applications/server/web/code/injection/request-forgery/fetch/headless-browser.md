---
title: "SSRF par navigateur headless et réutilisation de session"
description: "Un navigateur headless piloté par le serveur récupère des URL avec cookies, stockage et exécution de script, de sorte qu'une navigation choisie par l'attaquant atteint des hôtes internes, réutilise des sessions authentifiées et exécute du markup injecté dans un contexte privilégié."
keywords:
  - SSRF navigateur headless
  - Puppeteer
  - Playwright
  - service de capture d'écran
  - moteur de rendu PDF
  - réutilisation de session
  - endpoint de métadonnées
---

# Navigateur headless

Les services de capture d'écran, les exporteurs PDF, les dérouleurs de liens et les backends de scraping pilotent souvent un véritable moteur de navigateur (Puppeteer, Playwright, Selenium) depuis le serveur. Lorsque l'entrée de l'attaquant choisit la cible de navigation, ce navigateur est un client HTTP qui exécute aussi du JavaScript, transporte cookies et stockage, et charge des sous-ressources, ce qui en fait une primitive SSRF bien plus large qu'un simple fetch de bibliothèque.

## Atteindre des cibles internes

Une navigation de navigateur atteint les mêmes hôtes internes que n'importe quel SSRF, y compris les panneaux d'administration en loopback et l'endpoint de métadonnées cloud. Comme le résultat est généralement rendu (en capture d'écran ou en PDF), la réponse revient directement :

```
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://127.0.0.1:8080/admin
```

Le rendu incorpore le contenu récupéré dans l'image ou le document de sortie, donc même un service qui renvoie du texte brut ou du JSON est capturé sous forme de pixels, contournant la gestion du content-type.

## Fichiers locaux et script injecté

Un moteur headless honore souvent `file://`, et il exécute du markup. Si le service rend du HTML fourni par l'attaquant (un schéma courant dans les fonctionnalités HTML-vers-PDF), le script injecté s'exécute dans le contexte de la page et peut lire une ressource locale ou interne et la placer dans la sortie :

```html
<iframe src="file:///etc/passwd"></iframe>
<script>
fetch('http://127.0.0.1:2375/containers/json')
  .then(r => r.text()).then(t => { document.title = t; });
</script>
```

La capacité d'un `fetch` cross-origin à relire la réponse dépend des paramètres de sécurité du moteur ; un `<iframe>` ou un `<img>` qui se contente de rendre la cible dans la page n'a pas besoin d'une lecture cross-origin et fonctionne même lorsque les lectures par script sont bloquées.

## La réutilisation de session est l'escalade

La propriété dangereuse propre à ce client est la **session**. Un worker de rendu qui réutilise un seul profil de navigateur, ou à qui on remet les cookies d'un utilisateur pour capturer une vue authentifiée, transporte ces cookies. Le périmètre des cookies s'applique quand même : le navigateur n'envoie un cookie stocké qu'à une URL qui correspond à son `Domain` et son `Path`, donc cela escalade lorsque le profil détient un cookie valide pour la cible. C'est courant en pratique : un cookie de session délimité à un domaine d'entreprise parent (`.corp`) ou un cookie SSO couvre de nombreux hôtes internes sous lui, de sorte qu'une URL choisie par l'attaquant dans ce périmètre profite de l'identifiant :

```
http://internal-dashboard.corp/api/users/export
```

Là où le profil détient un cookie pour `internal-dashboard.corp` (ou un domaine parent qui l'inclut), le navigateur l'attache et la réponse rendue renvoie des données privilégiées. La frontière que cela abuse est la réutilisation d'un profil authentifié dont les cookies sont délimités à des hôtes internes atteignables tout en laissant l'URL de navigation influencée par l'attaquant ; un worker dont les cookies sont délimités uniquement à la cible externe prévue ne les fait pas fuir vers l'intérieur.

## Confirmer le client

Un comportement qui révèle un navigateur headless plutôt qu'un client [programmatique](programmatic.md) : l'exécution de `<script>` injecté, le chargement de sous-ressources et de redirections comme un navigateur, un user agent headless reconnaissable, et le rendu de réponses non-HTML en images. Une fois confirmé, investissez dans la réutilisation de session et l'exfiltration par script rendu plutôt que dans les payloads de scheme bruts. Les astuces d'hôte telles que le [DNS rebinding](../authority/domain-name/dns-rebinding.md) s'appliquent aussi ici, et franchissent la frontière d'origine du navigateur en plus de celle du serveur.

## Outils

- **Burp Suite** : soumettre des jobs de rendu et inspecter la capture d'écran ou le PDF renvoyé.
- **Burp Collaborator** : attraper les chargements aveugles de `file://` et de sous-ressources internes depuis le navigateur.
- **interactsh** : serveur OOB pour confirmer la navigation côté serveur et les callbacks de script injecté.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
