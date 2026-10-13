---
title: "SSRF par client HTTP programmatique"
description: "Les clients HTTP de bibliothèque récupèrent une URL influencée par l'attaquant sans moteur de navigateur. Le suivi de redirections, la gestion des schemes et les en-têtes par défaut décident jusqu'où une URL contrôlée par l'utilisateur peut atteindre."
keywords:
  - SSRF programmatique
  - axios
  - requests
  - HttpClient
  - suivi de redirections
  - URL contrôlée par l'utilisateur
  - requête sortante
---

# Programmatique

La plupart des sinks SSRF sont un appel HTTP de bibliothèque : `fetch`, `axios`, `got`, Java `HttpClient`, Python `requests`/`httpx`, Go `net/http`, Ruby `Net::HTTP`. Ceux-ci envoient la requête sans moteur JavaScript et généralement sans session interactive, donc la surface d'attaque est l'URL, la politique de redirection et de scheme du client, et les en-têtes que le code attache.

## Le sink central

La forme vulnérable est une valeur utilisateur affluant dans la cible de la requête :

```js
const r = await axios.get(req.query.url);   // url is attacker-controlled
```

```python
r = requests.get(user_url, timeout=5)
```

Pointé vers une adresse interne, le client la récupère et renvoie souvent le corps ou une erreur qui le reflète, donnant à l'attaquant une fenêtre sur le réseau interne.

## Le suivi de redirections élargit la portée

Les clients de bibliothèque diffèrent dans leur manière de suivre les redirections, et ce comportement est une surface de contournement. Une URL sur un hôte autorisé qui renvoie `Location: http://169.254.169.254/...` est suivie par un client qui n'a validé que la première URL. Le fait que les redirections soient suivies par défaut, le nombre de sauts autorisés et la possibilité pour le scheme de changer d'un saut à l'autre varient tous selon la bibliothèque :

- `requests` suit les redirections par défaut ; `axios` les suit ; Go `net/http` les suit par défaut avec une politique personnalisable.
- Certains clients rétrogradent ou refusent un changement de scheme (par exemple `https` vers `file`) lors d'une redirection ; d'autres non.

C'est la base du [contournement basé sur redirection](../query/bypassing-using-a-redirect.md) : valider le premier saut, se connecter sur un suivant.

## Gestion des schemes

Un client programmatique expose les schemes que son runtime enregistre. Un client Java construit sur `URL` peut honorer `file`, `ftp`, `jar` et `netdoc` ; les clients adossés à `curl` honorent `dict`, `gopher`, `ftp`, `tftp`, et davantage. Le sous-arbre [Scheme](../scheme/index.md) dépend entièrement de ce que le client spécifique accepte, donc énumérez les handlers enregistrés avant de vous engager sur un payload de scheme.

## Ce qu'il ne peut pas faire

Contrairement à un [navigateur headless](headless-browser.md), un client programmatique n'exécute pas le HTML ou le script renvoyé, et il ne transporte pas de session interactive à moins que le code n'attache explicitement des identifiants. Il n'y a donc pas d'exfiltration par script rendu, mais pas non plus de réutilisation automatique de cookies. Les gains sont la lecture des réponses internes, le scan via le timing de [Port](../authority/port.md), et l'interaction par octets bruts via un [Scheme](../scheme/index.md) capable. Les astuces d'hôte comme le [DNS rebinding](../authority/domain-name/dns-rebinding.md) s'appliquent parce que le client re-résout les noms.

## Outils

- **SSRFmap** : exploitation automatisée contre les sinks de clients HTTP de bibliothèque.
- **curl** : reproduire le fetch du client et tester directement la gestion des redirections et des schemes.
- **Burp Repeater** : rejouer la requête du sink en faisant varier l'URL, le scheme et les cibles de redirection.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
